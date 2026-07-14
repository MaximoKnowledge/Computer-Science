### Reviewer 1
1. We did not face major issues regarding this concern. As we sampled from it using torch's random library, which provided an efficient way of generating the points.
2. ?
3. The takeaway is that the model uses the statistic to model confidence, entropy and token distributions. The correcting phenomenon helps to explain the null results observed when performing the experiments at shallow layers, while for deep layers, as the model is not able to compensate the distortions we get the aforementioned results
4. The parabola-like geometry helps describe whether the model represents the same geometry across layers, shedding light into a model-level geometry that remains consistent across layers. To further assess the importance of this low-dimensionality, we conducted experiments steering the model on just a subspace of the mean vectors. The results were similar to the ones when doing the mean switching introduced in the work, which helped address the causal importance of the subspace
5. We mainly present the result as an outlier of the observations, as from the general view, most components keep aligned representations of the \tau subspace. This implies that what e.g. what the MLP represents as step 25, self-attention does as 75. This aids, but does not fully explains, why the correction takes place. As this opposite dynamic can help correct possible offsets that any of the two components generates



### Reviewer 2
1. We agree that the representation is non-trivial and may be useful for future work. Based on this observation we will provide a deeper proposal of future work, and stress the potential of exploiting this signal to develop new decoding methods
2. We agree that the high capacity of the network could potentially lead to a potential overfitting. Based on this feedback we decided to also train linear probes in a similar fashion (replacing the MLP by a 4096-d vector and using a sigmoid on-top). The results are similar to the ones in the MLP (degradation at the extreme layers and better performance at the middle ones), though with slightly lower R2 coefficients overall and a stronger degradation at the extremes. 

#### Comments and suggestions
1. For experiment we use the same probes as the ones showed in the R2 figures. We'll clarify this on the final version
2. We'll take this observation into consideration and update the work if we're able to do them on time. We emphasise though, that some phenomena may emerge only at scale, e.g. the correction mechanisms takes many layers to take place, and is not immediate


### Reviewer 3
1. We share some of the observations, though note that this has been explicitly addressed as future work in the discussion section
2. We agree on this view and may be better to improve the phrasing, although we note that in the extreme sense you're proposing, token-level would imply that the information is present at all tokens without having any interaction among them, which would further imply that the embeddings of the tokens would be sufficient to determine the timestep, and as the embeddings are always the same there would be no way of actually representing a \tau evolution, as they do not change when more tokens are unmasked. We used that specific phrasing as an alternative probe could read all the contextual information of the sequence by taking as input the whole sentence, this would trivialise the encoding, as the probe may just learn to count [MASK] and achieve perfect accuracy. By just training on one token, we show that it's enough to see a token's representation with the contextualised sequence-level information to decode \tau. We are open to further clarify this point if needed
3. The steering experiments solely had the objective of causally confirming the importance of the characterised \tau signal. However, we hypothesise that it can be used to steer the model when designing new unmasking algorithms, so we'll provide this observation inside the future works
4. We theorise that the final layer of LLaDA constructs an orthogonal representation as it's semantically different from all other layers as has been observed in autoregressive models. As for the Dream results, we do not fully see the weakening of the results, as the parabola-like and low-dimensionality structures are model-invariant. With this difference between LLaDA and Dream we provide further insight on how the models differ in their semantic encoding of \tau across layers
5. We note that the two analysed models are the most distributed ones in the DLM literature, and that while other models at the similar scale exist, they do no use the same diffusion formulation, which does not create a clear apples-to-apples comparison of the \tau encoding and would require a new work on unifying the time representation of these models (both at a theoretical and empirical level)

#### Comments and suggestions
1. We share this view and will update the notation to make it easier to follow and more consistent across the work (e.g. replacing \hat{\tau} by \hat{t}, as it's a discrete and not continuous signal)