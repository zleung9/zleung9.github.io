---
draft: false
title: "A simple neural network built from scratch with C (Part II)"
date: 2024-03-02
lastmod: 2024-11-29
tags: []
author: 
- Andrew Liang
description: 
summary: 
cover:
    image: "featured.png"
    alt: 
    relative: false
editPost:
    URL: 
    Text: 
showToc: true
disableAnchoredHeadings: false
---

In [Part I]({{< ref "/projects/neural-network-in-c-I" >}}) we  we discussed how to implement the forward calculation, in this article we will continue with the backpropagation.


## Gradient of output layer
Considering out simple model with the output layer $z$ and predicted probability $\hat{y}$. The last hidden layer is connected to $z$ via $w_2$. The gradient of loss function w.r.t. $w_2$ is [[ref](https://jmlb.github.io/ml/2017/12/26/Calculate_Gradient_Softmax/)]:
$$
\frac{\partial L}{\partial w_2}=h^T\times(\hat{y}-y)
$$

The second term $\hat{y}-y$ can be given by:
```c
void compute_output_gradient(float *grad_output, float *probability, int *labels, int batch_size) {
    for (int b; b<batch_size; b++) {
        for (int i; i<OUTPUT_SIZE; i++) {
            grad_output[b*OUTPUT_SIZE+i] = output[b*OUTPUT_SIZE+i];
        }
        grad_output[b*OUTPUT_SIZE + labels[b]] -= 1.0f; // y^_i - y_i
    }
}
```
## Gradient of bias and ReLu
Meanwhile the bias term $\partial L/\partial b$ and backward of `ReLu` function are simply:
```c
// bias backward
void bias_backward(float* grad_bias, float *grad, int batch_size, int size) {
    for (int i; i<size; i++) {
        grad_bias[i] = 0.0f;
        for (int b; b<batch_size; b++) {
            grad_bias[i] += grad[b*size+i]
        }
    }
}
// ReLU backward
void relu_backward(float *grad, float *x, int size) {
    for (int i = 0; i < size; i++) {
        grad[i] *= (x[i] > 0);
    }
}
```

## Loss backward
The total back propagation is given in the following block. This is essentially what `loss.backward()` does in the Pytorch implementation.

```c
void backward(MLP *nn, float *intput, float *hidden, float* output, int *labels, int batch_size) {
    //Initialize gradients to zero
    memset(nn->grad_weights1, 0, HIDDEN_SIZe*INPUT_SIZE*sizeof(float));
    memset(nn->grad_weights2, 0, HIDDEN_SIZe*INPUT_SIZE*sizeof(float));
    memset(nn->grad_bias1, 0, HIDDEN_SIZe*INPUT_SIZE*sizeof(float));
    memset(nn->grad_bias2, 0, HIDDEN_SIZe*INPUT_SIZE*sizeof(float));

    // Compute gradients for output layer
    float *grad_output = malloc(batch_size * OUTPUT_SIZE * sizeof(float));
    compute_output_gradients(grad_output, output, labels, batch_size);

    // Update gradients for weights2 (W2.grad = grad_output.T @ hidden)
    matmul_at_b(hidden, grad_output, nn->grad_weights2, batch_size, HIDDEN_SIZE, OUTPUT_SIZE);

    // Update gradients for bias2
    bias_backward(nn->grad_bias2, grad_output, batch_size, OUTPUT_SIZE);

    // Compute dX2 (gradient of loss w.r.t. input of second layer)
    float *dX2 = malloc(batch_size * HIDDEN_SIZE * sizeof(float));

    // grad_output @ W2.T = dX2 -> (B, 10) @ (10, 256) = (B, 256)
    matmul_a_bt(grad_output, nn->weights2, dX2, batch_size, OUTPUT_SIZE, HIDDEN_SIZE);

    // Compute d_ReLU_out (element-wise multiplication with ReLU derivative)
    float *d_ReLU_out = malloc(batch_size * HIDDEN_SIZE * sizeof(float));
    for (int i = 0; i < batch_size * HIDDEN_SIZE; i++) {
        d_ReLU_out[i] = dX2[i] * (hidden[i] > 0);
    }
    // retains its shape since its just a point-wise operation
    // Update gradients for weights1 (W1.grad = d_ReLU_out.T @ input)
    matmul_at_b(input, d_ReLU_out, nn->grad_weights1, batch_size, INPUT_SIZE, HIDDEN_SIZE);

    // Update gradients for bias1
    bias_backward(nn->grad_bias1, d_ReLU_out, batch_size, HIDDEN_SIZE);

    // Don't forget to release memory
    free(...);
    ...
}
```

## Update weights with tunable learning rate
The `optimizer.step()` essentially updates the weights with tunable learning rates. It can be written in the following function:
```c
void update_weights(NeuralNetwork *nn) {
    for (int i = 0; i < HIDDEN_SIZE * INPUT_SIZE; i++) {
        nn->weights1[i] -= LEARNING_RATE * nn->grad_weights1[i];
    }
    for (int i = 0; i < OUTPUT_SIZE * HIDDEN_SIZE; i++) {
        nn->weights2[i] -= LEARNING_RATE * nn->grad_weights2[i];
    }
    for (int i = 0; i < HIDDEN_SIZE; i++) {
        nn->bias1[i] -= LEARNING_RATE * nn->grad_bias1[i];
    }
    for (int i = 0; i < OUTPUT_SIZE; i++) {
        nn->bias2[i] -= LEARNING_RATE * nn->grad_bias2[i];
    }
}
```

Above is the complete guide to construct the forward and backward part of the training procedure. In the next article we will execute the training to see how much time it takes to complete the training.





