# HPC - Didacticiel d'utilisation d'OpenCL sur Taouey 

## Comment soumettre un job OpenCL avec SLURM
Afin de soumettre des tâches OpenCL à SLURM, nous aurons besoin principalement de deux code sources :

* (1) un programme OpenCL qui décrit la solution à votre problème et,
* (2) un script SLURM qui spécifie les ressources nécessaires, définit l'environnement et les commandes à exécuter.

# OpenCL par l’exemple
Commençons par tester une fonction simple, la fonction plot2d qui trace un ensemble de courbes en **2D**. Étant donné que **x** et **y** sont des vecteurs de même taille, **plot2d(x, y, <opt_args>)** trace le vecteur **y** en fonction du vecteur **x**. 

``` C
HelloWorld.cl:
__kernel void hello_kernel(__global const float *a,
                           __global const float *b,
                           __global float *result)
{
    int gid = get_global_id(0);
    result[gid] = a[gid] + b[gid];
}
HelloWorld.cpp:
int main(int argc, char** argv)
{
    cl_context context = 0;
    cl_command_queue commandQueue = 0;
    cl_program program = 0;
    cl_device_id device = 0;
    cl_kernel kernel = 0;
    cl_mem memObjects[3] = { 0, 0, 0 };
    cl_int errNum;
    // Create an OpenCL context on first available platform
    context = CreateContext();
    if (context == NULL)
    { cerr << "Failed to create OpenCL context." << endl;
return 1; }
// Create a command-queue on the first device available
// on the created context
commandQueue = CreateCommandQueue(context, &device);
if (commandQueue == NULL)
{
    Cleanup(context, commandQueue, program, kernel, memObjects);
    return 1;
}
// Create OpenCL program from HelloWorld.cl kernel source
program = CreateProgram(context, device, "HelloWorld.cl");
if (program == NULL)
{  Cleanup(context, commandQueue, program, kernel, memObjects);
  return 1;
}
// Create OpenCL kernel
kernel = clCreateKernel(program, "hello_kernel", NULL);
if (kernel == NULL)
{
    cerr << "Failed to create kernel" << endl;
    Cleanup(context, commandQueue, program, kernel, memObjects);
    return 1;
}
// Create memory objects that will be used as arguments to
// kernel. First create host memory arrays that will be
// used to store the arguments to the kernel
float result[ARRAY_SIZE];
float a[ARRAY_SIZE];
float b[ARRAY_SIZE];
for (int i = 0; i < ARRAY_SIZE; i++)
{
a[i] = i;
b[i] = i * 2; }
if (!CreateMemObjects(context, memObjects, a, b))
{
    Cleanup(context, commandQueue, program, kernel, memObjects);
    return 1;
}
// Set the kernel arguments (result, a, b)
    errNum = clSetKernelArg(kernel, 0,
                             sizeof(cl_mem), &memObjects[0]);
    errNum |= clSetKernelArg(kernel, 1, sizeof(cl_mem),
                             &memObjects[1]);
    errNum |= clSetKernelArg(kernel, 2, sizeof(cl_mem),
                             &memObjects[2]);
    if (errNum != CL_SUCCESS)
    {
        cerr << "Error setting kernel arguments." << endl;
        Cleanup(context, commandQueue, program, kernel, memObjects);
        return 1;
}
    size_t globalWorkSize[1] = { ARRAY_SIZE };
    size_t localWorkSize[1] = { 1 };
    // Queue the kernel up for execution across the array
    errNum = clEnqueueNDRangeKernel(commandQueue, kernel, 1, NULL,
                                    globalWorkSize, localWorkSize,
                                    0, NULL, NULL);
    if (errNum != CL_SUCCESS)
    {
        cerr << "Error queuing kernel for execution." << endl;
        Cleanup(context, commandQueue, program, kernel, memObjects);
        return 1;
}
    // Read the output buffer back to the Host
    errNum = clEnqueueReadBuffer(commandQueue, memObjects[2],
                            CL_TRUE, 0, ARRAY_SIZE * sizeof(float),
                            result, 0, NULL, NULL);
    if (errNum != CL_SUCCESS)
    {
        cerr << "Error reading result buffer." << endl;
        Cleanup(context, commandQueue, program, kernel, memObjects);
        return 1;
}
    // Output the result buffer
    for (int i = 0; i < ARRAY_SIZE; i++)
    {
        cout << result[i] << " ";
    }
    cout << endl;
    cout << "Executed program successfully." << endl;
    Cleanup(context, commandQueue, program, kernel, memObjects);
return 0;
}
```
Sauvegardez ce code dans un fichier nommé **test_OpenCL.c**.

## Écriture du script SLURM 
Maintenant, vous devez créer un script SLURM **test_OpenCL.sh** pour soumettre votre travail. Voici un exemple de script SLURM :

``` C
#!/bin/bash
#SBATCH --job-name=test_OpenCL
#SBATCH --output=test_OpenCL.out
#SBATCH --error=test_OpenCL.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=1:00:00

# Chargement du module OpenCL
module purge
module load OpenCL/3.0/OpenCL-3.0

# Compile the C program with OpenCL
gcc -o test_OpenCL test_OpenCL.c -lOpenCL

# Exécution du script OpenCL
./test_OpenCL
```

Pour exécuter le code OpenCL, soumettez simplement le travail à SLURM avec la commande suivante :
```
$ sbatch ./test_OpenCL.sh
```

# Suivi des job OpenCL sur SLURM
Afin de suivre l'état d'avancement de votre job une fois soumis, se référer au didacticiel sur SLURM.  
cat("[Didacticiel sur SLURM](https://github.com/DiopBabacarEdu/TaoueY-HPC/tree/main/SLURM)")


