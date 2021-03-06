# Use with GPUs

Note that the differential equation solvers will run on the GPU if the initial
condition is a GPU array. Thus for example, we can define a neural ODE by hand
that runs on the GPU (if no GPU is available, the calculation defaults back to the CPU):

```@example gpu
using DifferentialEquations, Flux, Optim, DiffEqFlux, DiffEqSensitivity

model_gpu = Chain(Dense(2, 50, tanh), Dense(50, 2)) |> gpu
p, re = Flux.destructure(model_gpu)
dudt!(u, p, t) = re(p)(u)

# Simulation interval and intermediary points
tspan = (0.0, 10.0)
tsteps = 0.0:0.1:10.0

u0 = Float32[2.0; 0.0] |> gpu
prob_gpu = ODEProblem(dudt!, u0, tspan, p)

# Runs on a GPU
sol_gpu = solve(prob_gpu, Tsit5(), saveat = tsteps)
```

and `concrete_solve` works similarly. Or we could directly use the neural ODE
layer function, like:

```
prob_neuralode_gpu = NeuralODE(gpu(dudt2), tspan, Tsit5(), saveat = tsteps)
```
