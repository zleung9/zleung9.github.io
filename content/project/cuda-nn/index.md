---
draft: false
title: A simple neural network built from scratch written in C
date: 2024-11-01
---

# Components of a neural network

To quickly construct the neural network structure, I will use the Pytorth to construct a simple neural network for demonstration purpose.  I purposely left out batchnorm, residual blocks, lower-precision, and other optimizations to keep the code simple and easy to understand.
The structure is simple. It contains two fully connected layers with a ReLu activation function in between. First I will implement this class in C language, and then turn it into CUCA enabaled code. 
```Python
class MLP(nn.Module):
    def __init__(self, in_features, hidden_features, num_classes):
        super(MLP, self).__init__()
        self.fc1 = nn.Linear(in_features, hidden_features)
        self.relu = nn.ReLU()
        self.fc2 = nn.Linear(hidden_features, num_classes)

    def forward(self, x):
        x = x.reshape(batch_size, 28 * 28)
        x = self.fc1(x)
        x = self.relu(x)
        x = self.fc2(x)
        return x


model = MLP(in_features=784, hidden_features=256, num_classes=10).to("cuda")
# model = torch.compile(model)
optimizer = optim.SGD(model.parameters(), lr=learning_rate)
```

In advanced frameworks such as Pytorch, we often see the use of `optimizer` or `loss.backward()`. What do they do? Well, to find out what is under the hood, we need to dig into what a basic training loop does. In a nutshell, a typical training cycle consists of forward calculation, loss calculation, back propagation and updating weights. 

This procedure will be repeated again and again through epochs, and for each epoch through batches. The pseudo Python code below describes how it works. 

```Python
for epoch in range(epochs):
    for X, y in train_dataloader:
        pred = model.forward(X) # forward calculation
        loss = loss_function(pred, y) # calculate loss
        gradient = y_pred - y # calcualt gradient
        model.backward(gradient) # backward propogation
        model.update_gradient() # update the gradient
    
    X_val, y_val = next(iter(val_dataloader))
    pred_val = model.forward(X_val)
    loss = loss_function(pred_val, y_val)

X_test y_test = test_dataloader
pred_test = model(X_test)
loss = loss_function(pred_test, y_test)
```

- forward calculation. This is the basic execution of  the model. It is essentially matrix mulplication in this case. So we need to write our function to perform matrix mutiplication.
- loss calculation. In this case we will use write a cross-entropy loss function.
- back propagation. It further breaks down to two parts: gradient calculation and gradient update.
- update weights. 


# Components of a neural network from scratch

First of all, let's define the structure of the neural network. We will take the simp 
```c
typedef struct {
    float *weights1;
    float *weights2;
    float *bias1;
    float *bias2;
    float *grad_weights1;
    float *grad_weights2;
    float *grad_bias1;
    float *grad_bias2;
} MLP;
```


Let's write down the building block of the forward function. The fully connected layer is essentially matrix multiplication with weights plus bias term {{< math >}}$ \textbf{X}\times\textbf{W}+\textbf{B} ${{< /math >}}. Since in the backpropagation the weight term and bias term are handled differently, we will implement them seperately.

In the code below we represent 2D matrix using a flattened 1D array. The implimentation only works in seeries, which significantly slows down the computation. This can be optimized using CUDA.

```C
// A has shape m x n, B has shape n x k and C has shape m x k
void matmul_a_b(float *A, float *B, float *C, int m, int n, int k) {
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < k; j++) {
            C[i * k + j] = 0.0f;
            for (int l = 0; l < n; l++) {
                C[i * k + j] += A[i * n + l] * B[l * k + j];
            }
        }
    }
}

// X has size (batch_size, n), bias is a 1D array of size n.
void bias_forward(float *X, float *bias, int batch_size, int n) {
    for (int b=0; b<batch_size>; b++) {
        for (int i=0; i<n; i++) {
            x[b*n + i] += b[i];
        }
    }
}
```

We also need to define {{< math >}}$ \textbf{A}\times\textbf{B}^T ${{< /math >}}, and {{< math >}}$ \textbf{A}^T\times\textbf{B} ${{< /math >}}. It is just a matter of changing the iteration above. 

Here comes the `ReLu` layer, along with the last layer `softmax` function.

```c
// input x has size `n`. Assign zero if the value is negative.
void relu(float *x, int n) {
    for (int i=0; i<n; i++) {
        x[i] = fmaxf(0.0f, x[i]);
    }
}

// input x has size n. 
void softmax(float *x, int batch_size, int n) {
    for (int b = 0; b < batch_size; b++) {
        float sum = 0.0f;
        for (int i = 0; i < size; i++) {
            x[b * size + i] = expf(x[b * size + i]);
            sum += x[b * size + i];
        }
        for (int i = 0; i < size; i++) {
            x[b * size + i] = fmaxf(x[b * size + i] / sum, 1e-7f);
        }
    }
}
```

 We now construct the forward function of the model.
 ```c
 void forward(MLP *nn, float *X, float *hidden, float *output, int batch_size) {
    // Input to Hidden: X @ W1
    matmul_a_b(X, nn->weight1, hidden, batch_size, INPUT_SIZE, HIDDEN_SIZE);
    // Add bias1
    bias_forward(hidden, nn->bias1, batch_size, HIDDEN_SIZE);
    // Apply ReLU
    relu_forward(hidden, batch_size * HIDDEN_SIZE);
    // Hidden to Output (hidden @ W2)
    matmul_a_b(hidden, nn->weight2, output, batch_size, HIDDEN_SIZE, OUTPUT_SIZE);
    // Add bias2
    bias_forward(output, nn->bias2, batch_size, OUTPUT_SIZE);
    // Apply softmac
    softmax(output, batch_size, OUTPUT_SIZE);
 }
 ```

 ## Calculate loss function

 ```c
 float cross_entropy_loss(float *output, int *labels, int batch_size) {
    float total_loss = 0.0f;
    for (int b = 0; b < batch_size; b++) {
        total_loss -= logf(fmaxf(output[b * OUTPUT_SIZE + labels[b]], 1e-7f));
    }
    return total_loss / batch_size;
}
 ```

 











# C implementation



# Improvement over time.