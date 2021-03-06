---
keywords: fastai
description: "My thoughts about anomaly detection using GAN's"
title: "Anomaly Detection Using Generative Adversarial Networks"
toc: false
badges: true
comments: true
categories: [thesis, gan, deep learning]
image: "images/anomaly/header.png"
hide: false
search_exclude: false
nb_path: _notebooks/2019-04-20-anomaly-detection-using-bigans.ipynb
layout: notebook
---

<!--
#################################################
### THIS FILE WAS AUTOGENERATED! DO NOT EDIT! ###
#################################################
# file to edit: _notebooks/2019-04-20-anomaly-detection-using-bigans.ipynb
-->

<div class="container" id="notebook-container">
        
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>In this post I will explain the architecture of the bigan and how can it be used for the anomaly detection problem. The 
papers that inspired this post are down below at the references section.</p>
<h1 id="What-is-Anomaly-Detection-?">What is Anomaly Detection ?<a class="anchor-link" href="#What-is-Anomaly-Detection-?"> </a></h1><p>Anomaly detection is one of the most important problems concerning multiple domains including manufacturing, 
Cyber-security, fraud detection and medical imaging. At its core an Anomaly Detection method should learn the data 
distribution of the normal samples which can be complex and high dimensional to identify the anomalous ones.</p>
<p>The method I will explain focuses on the reconstruction based approach to indentify the anomalous samples. By learning the 
data distribution and its representation, model is then able to reconstruct a sample image that is similar to the input for 
the inference. By defining a score function to measure the similarity between the input image and the reconstructed output, 
we can attain a score that can be used to identify a sample as anomalous or normal. Since the model is trained with the normal 
images, ideally, when the test image is normal, the reconstructed sample is expected to obtain a lower anomaly score compared to 
an anomalous image. That is the basis for the anomaly detection using reconstruction based approach.</p>
<p>Generative Adversarial Networks, or GANs are considered as a significant breakthrough in deep 
learning. They are used to model complex and high dimensional distributions using adversarial training. Let's explore 
what that means and how we can use it 
for this problem.</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h1 id="Intuition-Behind-GANs">Intuition Behind GANs<a class="anchor-link" href="#Intuition-Behind-GANs"> </a></h1><p>Generative Adversarial Networks consist of 2 networks, one generator and one discriminator. The generator is responsible 
 for generating sample images that is similar to the dataset samples and tries to fool the discriminator. The purpose of 
 the discriminator is to identify whether the image is from the dataset or it is generated by the generator, in other words to 
 classify input images as "real" or "fake".</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p><img src="/images/copied_from_nb/images/anomaly/gan.jpg" alt="The pipeline of GAN Framework"></p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>The main training idea behind GANs is based on game theory and assuming that the two network are competing each other. 
The model is usually trained with the gradient-based approaches by taking minibatch of fake images generated by transforming 
random vectors sampled from $p_z(z)$ via the generator and minibatch of data samples from $p_{data}(x)$. They are 
used to maximize $V(D, G)$ with respect to parameters of $D$ by assuming a
constant $G$, and then minimizing $V(D, G)$ with respect to parameters of $G$ by assuming a constant $D$.</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
$$
\normalsize
\min _{G} \max _{D} V(D, G)=\mathbb{E}_{\boldsymbol{x} \sim p_{\text { data }}(\boldsymbol{x})}[\log D(\boldsymbol{x; \theta_d})]+\mathbb{E}_{\boldsymbol{z} \sim p_{\boldsymbol{z}}(\boldsymbol{z})}[\log (1-D(G(\boldsymbol{z; \theta_g})))]
$$
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>As we can see, there are 2 loops and two terms. Let's disect each term. They will provive useful for the BiGAN model in the 
 next section.</p>
<h2 id="Term-1">Term 1<a class="anchor-link" href="#Term-1"> </a></h2>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
$$
\begin{aligned}
    D(\boldsymbol{x ; \theta_d}) &amp;\rightarrow \text{Likelihood of Discriminator identifying x as Real}\\
    \\
    \log D(\boldsymbol{x ; \theta_d}) &amp;\rightarrow \text{Log Likelihood of Discriminator identifying x as Real} \\
    \\
    \mathbb{E}_{\boldsymbol{x; \theta_d} \sim p_{\text { data }}(\boldsymbol{x})}[\log D(\boldsymbol{x; \theta_d})] &amp;\rightarrow \text{Expected Log Likelihood of input samples from real data} \\
\end{aligned}
$$
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h2 id="Term-2">Term 2<a class="anchor-link" href="#Term-2"> </a></h2>
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
$$
\begin{aligned}
    G(\boldsymbol{z ; \theta_g}) &amp;\rightarrow \text{Generated image sample from noise $z$} \\
    \\
    D(G(\boldsymbol{z ; \theta_g}); \theta_d) &amp;\rightarrow \text{The likelihood of the image from Generator being real} \\
    \\
    \log D(G(\boldsymbol{z ; \theta_g}); \theta_d) &amp;\rightarrow \text{The log likelihood of the image from Generator being real} \\
    \\
    \log (1-D(G(\boldsymbol{z ; \theta_g}); \theta_d)) &amp;\rightarrow \text{The log likelihood of the image from Generator being fake} \\
    \\ 
    \mathbb{E}_{\boldsymbol{z } \sim p_{\boldsymbol{z}}(\boldsymbol{z})}[\log (1-D(G(\boldsymbol{z ; \theta_g}); \theta_d))] &amp;\rightarrow \text{E.L.L of Discriminator identifying generated image as fake}\\
\end{aligned}
$$
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>Now there are two main loops in the equation that we need to examine. The inner loop of the discriminator and the 
outer loop of the Generator. 
If want to summarize their logic:</p>
<p>Inner loop is the training objective of the discriminator. It wants to maximize its probability of 
classifying an image as real or fake. So It needs to maximize $D(x)$ to classify real images ( since $D(x)$ is the 
probability of a discriminator identifying image as real) and it also needs to maximize $(1 - D(G(z))$ to maximize its 
probability for spotting fake images.</p>
<p>The outer loop is the training objective of the generator. Since it's only on one of the terms we don't 
need to look at the first term. Generator wants to fool the discriminator by generating more real like sample images. So 
in order for it to maximize its probability to get classified as real, it needs to minimize discriminator's probability 
of classifying generated image as fake. So it needs to minimize $D(G(z))$ to increase $(1 - D(G(z)))$.</p>
<p>Now that's out of the way we can focus on what is BiGAN and how does it work ?</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<h1 id="BiGAN-Architecture">BiGAN Architecture<a class="anchor-link" href="#BiGAN-Architecture"> </a></h1><p>BiGAN is composed of a standard GAN with an additional Encoder that is simultaneously trained with the generator and discriminator.
This encoder learns the mapping representation from the input image to the latent space representation (noise). This approach enables
inference process much faster than the previous proposed method which is an iterative optimization process via backpropagation for 
each sample.</p>
<p><img src="/images/copied_from_nb/images/anomaly/bigan.jpg" alt="BiGAN Framework"></p>
<p>With the addition of the Encoder, the Discriminator behavior changes a little. Now the discriminator not only discriminates in data
space ($z$ or $G(z)$) but jointly in data and latent space tuples ($x$, $E(x)$) versus ($G(z)$ , $z$). Generator and encoder are trying 
to fool the discriminator. 
&lt;/p&gt;</p>
<p>Let's explain the objective function of the BiGAN like we did with GAN.</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>$
\min _{G,E} \max _{D} V(G, D, E)=\mathbb{E}_{x \sim p_{x}}[\log D(x, E(x))]+\mathbb{E}_{z \sim p_{z}}[\log (1-D(G(z), z))]
$</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>Our objective function is similar to GAN with the difference of the noise tuple from the encoder and the latent space
distribution. From the discriminator's perspective, the pair with input image and the encoded noise of the input image 
should be classified as real, and the tuple with sampled noise and the generated image should be classified as fake. 
So again, it needs to maximize the probability of discriminating $(x, E(x))$ 
tuple as real and $(G(z), z)$ tuple as fake.</p>
<p>We can consider the encoder and generator loss in the same loop because they are both trying to fool the discriminator. 
Encoder wants to minimize discriminator's probability of classifiying $(x, E(x))$. The reason for this is that in order 
for encoder to be an optimal one, it needs to learn the invert the input from the true data distribution, to act as 
$E = G^{-1}$. Generator again wants to minimize Discriminator's ability to spot a fake image so consequentially it wants to 
maximize $D(G(z), z)$ probability.</p>
<p>After the training of the model is done, we can make an inference to test the model. The score function $A(x)$ is composed of the 
combination of the reconstruction loss ($L_G$) and discrimination-based loss ($L_D$). The score function and its components are depicted
below.</p>

</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
$$
\begin{aligned}
A(x) &amp;= \alpha L_{G}(x)+(1-\alpha) L_{D}(x) \\
\\
L_{G}(x) &amp;= \|x-G(E(x))\|_{1} \\
\\
L_{D_{1}} &amp;= \sigma(D(x, E(x)), 1) \\
\\
L_{D_{2}} &amp;= \left\|f_{D}(x, E(x))-f_{D}(G(E(x)), E(x))\right\|_{1} \\
\end{aligned}
$$
</div>
</div>
</div>
<div class="cell border-box-sizing text_cell rendered"><div class="inner_cell">
<div class="text_cell_render border-box-sizing rendered_html">
<p>Reconstruction loss measures the difference between the input image and the reconstructed image. 
There are 2 types of discrimination loss that we can define. First one depends on the 
sigmoid cross entropy loss from the discriminator of x 
being a real example (class 1 in this case), and the second method for defining the discriminator loss is based on 
the <em>feature matching</em> loss with $f_D$ returning the layer preceding the logits for the given inputs in the discriminator. 
This loss evaluates if the reconstructed data has similar features in the discriminator as the true sample. 
Samples with larger $A(x)$ values are considered as more likely from the 
anomalous sample.</p>
<p>In this introductory post I wanted to talk about the BiGAN architecture for the anomaly detection task. 
In the following posts I will talk more about anomaly detection, different architectures that can be used with more 
detailed summary of the architectures and the training techniques.</p>
<h1 id="References">References<a class="anchor-link" href="#References"> </a></h1><ul>
<li><a href="https://arxiv.org/abs/1605.09782">Adversarial Feature Learning</a></li>
<li><a href="https://arxiv.org/abs/1606.00704">Adversarially Learned Interface</a></li>
<li><a href="https://arxiv.org/abs/1802.06222">Efficient GAN Based Anormaly Detection</a></li>
</ul>

</div>
</div>
</div>
</div>
 

