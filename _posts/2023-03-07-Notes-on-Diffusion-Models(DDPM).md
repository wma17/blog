---
layout: post
title: Notes on Diffusion Models(DDPM)
subtitle: Notes to help me understand the math in the paper
readtime: true
tags: [Diffuusion Models]
comments: true
---

<!-- gh-repo: daattali/beautiful-jekyll
gh-badge: [star, fork, follow]
tags: [test] -->

<script type="text/javascript" src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=default"></script>

Two weeks ago, a friends recommended me the [ControlNet](https://arxiv.org/pdf/2302.05543.pdf), hugely impressed by the high quality generated image, I decided to dive into the Diffusion Models and review the past significant works. So this is the beginning, it's not easy for me to go through all the details contained, but I will try my best to present it clearly.

# Notes on Diffusion Models

The idea of Diffusion Models comes from non-equilibrium thermodynamics, generally speaking, we slowly add random Gaussian noise to the data step by step, then learn to reverse the procedure to construct new satisfactory data. Imagine that when the forward steps are very long, the data will completely be a noise; So this model contains the potential to generate a new image by denoising a Gaussian noise step by step. This potential becomes a fact after experiments. 

![process](https://github.com//wma17/images/2023-03-07-DDPM/process.png)

## Forward Process

Let's take it more specifically, we take $\mathrm{x_0}$ from the real data distribution $q(\mathrm{x})$, that is, $\mathrm{x_0} \sim q(\mathrm{x})$. Now the ***Forward Process*** means we add small amount of Gaussian noise step by step, and finally we get a sequence $\mathrm{x_1,x_2,...,x_T}$ after $T$ steps. We introduce the variance schedule $\left\{ \beta_t \right\}_{t=1}^{T}$ to control the stepsize, where $\beta_t\in(0,1)$ for $t=1,2,...,T$. And we get
$$
\begin{equation*}
q(\mathrm{x_t|x_{t-1}})=\mathcal{N}(\mathrm{x_t},\sqrt{1-\beta_t}\mathrm{x_{t-1}},\beta_t\mathrm{I}),\quad q(\mathrm{x_{1:T}|x_0})=\prod_{t=1}^Tq(\mathrm{x_t|x_{t-1}}),
\end{equation*}
$$
where ${\mathcal{N}(\mathrm{x_t},\mu,\epsilon)}$ means a Gaussian$(\mu,\epsilon)$ with output being $\mathrm{x_t}$, and $q(\mathrm{x_{1:T}|x_0})$ implies a joint distribution(a trajectory) of $\mathrm{x_1,...,x_T}$ given $\mathrm{x_0}$. 

## Reverse Process

The ***Reverse Process*** is to denoise from $\mathrm{x_T}$, if we can sample from $q(\mathrm{x_{t-1}|x_t})$, then we can get the specific $\mathrm{x_0}$ from a random Gaussian noise. When the stepsize $\beta_t$ is very small, we have 
$$
\begin{equation*}
q(\mathrm{x_t|x_{t-1}})=\mathcal{N}(\mathrm{x_t},\sqrt{1-\beta_t}\mathrm{x_{t-1}},\beta_t\mathrm{I}) \approx\mathcal{N}(\mathrm{x_t},\mathrm{x_{t-1}},\beta_t\mathrm{I}),
\end{equation*}
$$
let $\beta_t\mathrm{I}=\epsilon$, we obtain
$$
\begin{equation*}
\mathrm{x_t}=\mathrm{x_{t-1}}+\epsilon  \  \rightarrow \ \mathrm{x_{t-1}}=\mathrm{x_t}-\epsilon,
\end{equation*}
$$
which implies $q(\mathrm{x_{t-1}|x_t})\sim\mathcal{N}(\mathrm{x_{t-1}},\mathrm{x_t},\epsilon)$, a normal distribution. Especially when the number of the steps is large, we can have small enough $\beta_t$s to approach this approximation. However, to calculate $q(\mathrm{x_{t-1}|x_t})$ needs the whole sequence. Here we choose a model $p_\theta(\mathrm{x})$ to approximate $q(\mathrm{x})$. We have
$$
\begin{equation*}
p_\theta(\mathrm{x_{t-1}}|\mathrm{x_{t}})\sim\mathcal{N}(\mathrm{x_{t-1}},\mu_\theta(\mathrm{x_t},t),\Sigma_\theta(\mathrm{x_t},t)),\quad 
p_\theta(\mathrm{x_{0:T}})=p_\theta(\mathrm{x_T})\prod_{t=1}^Tp_\theta(\mathrm{x_{t-1}}|\mathrm{x_{t}}).
\end{equation*}
$$
