---
layout: post
title: Attention Speed Up
subtitle: 
gh-repo: honky-tonk/honky-tonk.github.io
gh-badge: [star,follow]
tags: [Stat]
comments: true
---

<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>


# What is Attention?

"Attention is all you need" come from transformer paper, before we get start we should know what is transformer? you could click this [url](https://jalammar.github.io/illustrated-transformer/) to get base knowledge about transformer

Next we will talk about how to generate token, when we prompt a sentence, how QKV work in inference 

In mathematica, we could make a prompt to a conditional probability formula
$$P(Token_3)=P(Token_1)P(Token_2|Token_1),P(Token_4)=P(Token_1)P(Token_2|Token_1)P(Token_3|Token1, Token2)...$$

In **Training** we could input token 1 and use this formula above to predict token2, then we compare token 2 we predict and token 2 which realistic, then backforward to update the parameter

In **Inference** we already know the 10 token we prompt, so we could use this same formula
$$P(Token_{11}|Token1,...,Token_{10})$$

From the View of Transformer， in inference we could know, the front of 10 token Q vector and K vector(they group of a Q matrix and  k metrix), then we mask futher knowledge, then softmax to probability space, then random Dropout the parameter, finally multiply V matrix $$O=Dropout(Softmax(Mask(QK^T)))V$$ , then we know next token's probability distribution then we pick the highest one to output

# Why we need Attention Speed Up
Attention compute need large DRAM(or HBM in Morden GPGPU) SRAM resource, Intuitively the SRAM is faster then DRAM, but SRAM Space is more precious then DRAM, and initlly the Q/K/V matrix(for all input token vector) and S matrix(which result of Softmax), O matrix(output matrix) store in global memory(SRAM), when we run inference or training there is the step of attention work in GPU memory system

1. store all  **Q**, **K**, **V** matrix, **S** matrix(Pre-allocate), **P** matrix(Pre-allocate), **O** matrix(Pre-allocate) in Global Memory
2. Fetch Q, K matrix to SRAM
3. compute Q @ K^T then we got S matrix in SRAM
4. store S matrix to DRAM(the duplicate of S matrix still in SRAM) and fill the S matrix we allocate before
5. compute Softmax for S matrix then we got P matrix and store P matrix(duplicate of P matrix still in SRAM) to Global Memory we allocate before
6. in this step we will clean Q and K matrix we fetched in SRAM, and fetch V matrix O matrix(we will cover this matrix later) to SRAM, then compute P @ V^T then we get O matrix(cover the SRAM space we fetch before), finally we store the O matrix back to DRAM


in normal case of attention exec in GPGPU we illustrate before, we could find this data(matrix) ldst(load/store) between frequently, we need the high performance way to handle attention

# Flash Attention
Before we dive into flash attention, we need to known online softmax which nvidia engineer propose in 2018, the common softmax formula list below

$$SoftMax(V_i)=\frac{e^{V_i}}{\sum_{j=1}^ne^{V_j}}$$

which

$ V_i$ is the $i_{th}$ element of vector V

so if we got a vector like $[3, 2, 5, 7]$ then we could use softmax function to handle each element of vector,like $[softmax(3), softmax(2), softmax(5), softmax(7)]$ 
> sigmoid vs softmax, sigmod formula is $sigmoid(V_i)=\frac{1}{e^{-z}}$, sigmoid is use for Binary classification, but softmax used for muliti classification, if we use softmax in multi classification we could find all result sum together is 1 but sigmoid

we could write naive softmax with pytorch
```python
# input (X, X) matrix
# output ()
def naive_softmax(x: torch.Tensor)->torch.Tensor:
    x_max       = x.max(dim=1)[0] #get all row max element
    safe_x      = x - x_max.view(-1, 1) # -1 mean calculate dimension automatically, base other paramater
    numerator   = torch.exp(safe_x)
    denominator = numerator.sum(dim=1)
    sm_out      = numerator / denominator.view(-1, 1)
    return sm_out
```
the naive softmax is low performence, cause naive softmax need loop matrix twice, once get all max element each row, next do exponential function for matrix 

How Online softmax work? in online softmax exponential function running in first loop(find max element), the key problem is how we find the relation between old normalizer(with old row_max) and final normalizer(with correct row_max), we derivative formula and  prove it, the online_softmax code list below
```python
def online_softmax(x: torch.Tensor)->torch.Tensor:
    row_count, col_count = x.shape
    output = torch.zeros_like(x)
    # search row
    for r in range(row_count):
        row_max    = 0
        normalizer = 0
        # search col
        for c in range(col_count):
            curr = x[r,c]
            prev_old_max = row_max
            row_max = max(row_max, curr)
            if row_max > prev_old_max:
                print(f"updated row max is now {row_max}, row ={r}")
            normalizer = normalizer * torch.exp(prev_old_max - row_max) + torch.exp(curr - row_max)
        output[r,:] = torch.exp(x[r,:] - row_max) / normalizer
    return output
```

suppose we got prev old max $m_{t-1}$, old normalizer $normalizer_{t-1}$, the max element of row is $m_t$, and $normal=\sum_i^te^{x_i-m}$, then $$norm_{t-1}=\sum_{i=1}^{t-1}e^{x_i-m_{t-1}} \newline norm_{t-1}e^{m_{t-1}-m}=\sum_{i=1}^{t-1}e^{x_i-m_{t-1}}.e^{m_{t-1}-m} \newline norm_{t-1}e^{m_{t-1}-m}=\sum_i^{t-1}e^{x_i-m} \newline norm_{t-1}e^{m_{t-1}-m} + e^{x_i-m}=\sum_i^{t-1}e^{x_i-m} + e^{x_i-m}=\sum_i^{t}e^{x_i-m}=norm_t$$

so we could use this formula directly in code below
```python
normalizer = normalizer * torch.exp(prev_old_max - row_max) + torch.exp(curr - row_max)
```
Back to Flash Attention, the two keys consept is **tiling** and **recomputation** 

the attention formula in sample list above

$O=Softmax(QK^T)*V$

Assumption
- $S = QK^T$


In tiling we need split **S row vector** and **V col vector** to muliti GPU Cache block then we use **Online softmax** to compute softmax, then we compute v col vector to a new row   

For Recomputation, We do not store the intermediate activeations in SRAM but recomputation, like score we do not need store score in SRAM for backpropagation but recomputation it.

# Paged Attention

>the Key concept of paged attention is how to use KV Cache efficient to maximize token throughput

How did KV Cache work ?

In the nutshell, KV Cache store in SRAM, why KV Cache not Q cache? Most of LLM is autoregressive model that input token generation by itself untill model read a EOL then stop to generation, we could simulate the inference stage of autoregressive model.
1. we prompt 3 token which is "I Love Learn" to LLM 
2. LLM embeding three token to vector and add their position eg. I embeding to vector [1.2, 2.3, 1.6] and this token position is [1.3, 0.1,0.3] final we add them together to a new vector which include position and characteristic 
3. After we have this three token's vector, LLM map three token's vector to 3 group vector (which combine to 3 matrix) Q, K, V respectively, then we calculate Softmax(Q.K)V and  through fnn(maybe normalization first) then we generation the next token 
4. This is most important stage for inference workload, the token we generation feedback to LLM(This is what autoregresive model look like), In stage-3 we store the K, V matrix of each 3 token to SRAM, in the newest token (4th) we get their K, V vector and add to SRAM KV Cache Space, we don't need to store Q Cache to the SRAM, **because the result is output already(former token which generate)**, we don't need store former token's Q vector but newest token's Q vector, then dot product new K matrix then softmax and dot product with V matrix...
5. We feed the output token back to LLM, then looooooop.  

In inference workload KV Cache is necessary, but we need allocate multi kv cache space for each request, how many kv cache space we need allocated for each token in one request? there is formula for 13B paramater model 

2(key and value vectors) * 5120(hidden state size) * 40(number of layer) * 2(bytes per
FP16) = 800KB

it's seem not much for our GPU DRAM, but if we have 1000 prompt request same time each request's context length have 100 token, we need allocate 800KB * 1000 * 100 =  76GB space for KV cache, but there are some problem here, 
1. Internal-fragmentation: due to unknown output length, we would like to pre-allocate a large space for KV cache(most of time the token we generation not use all space which pre-allocate before)
2. Reservation: a prompt request will reserve many space, this free space not for current token but future model generate, so we could find there are many non-continue space in kv cache
3. External Fragmentation: Because many prompt request target of token generation number is different, so in KV cache could have many small Fragmentation between large request reservation and small request reservation which can not allocate to other request!

So How Vllm solve this problem? like OS alway do, Vllm seperate DRAM to multi block(same as page in os memory, in vllm each block has 16 or 32 token), In Vllm two type block is presented, logical block(same as virtual page in os) and physical block(same as physical page in os), each request has their own logical block then through block table map to physical block, eg. we got two request A and B, A use 2.5 logical block, B use 3 logical block, through block table they are mapped to 6 non-continue physical block space(this space is not large like before), there no external fragmentation anymore! but internal fragmentation(they exist in a block), and the block allocation is on-demand which mean request A or request B really need 4th block  for KV accelerate, then 4th block allocation is start

In other workload(not chatbot) like code generation, if one prompt have two or more type output that you need pick one, in this scenario different output could share one physical block(cause the most piece of  output sentence or code is same)

![LogicalBlockMapToPhysicalBlock](https://honky-tonk.github.io/assets/img/2025031/LogicalBlockMapToPhysicalBlock.png)

![TwoLogicalBlockSharingOnePhysicalBlock.png](https://honky-tonk.github.io/assets/img/2025031/TwoLogicalBlockSharingOnePhysicalBlock.png)

# How Sglang work

