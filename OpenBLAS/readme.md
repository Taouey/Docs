# HPC - Didacticiel d'utilisation d'OpenBLAS sur Taouey 

## Comment soumettre un job OpenBLAS 
Afin de soumettre des tâches OpenBLAS à SLURM, nous aurons besoin principalement de deux code sources :

* (1) un script OpenBLAS qui décrit la solution à votre problème et,
* (2) un script SLURM qui spécifie les ressources nécessaires, définit l'environnement et les commandes à exécuter.
Nous utilisons gcc dans cet exemple pour l'impémentation à OpenBLAS.

#  Multiplication de matrices sur OpenBLAS
Voici un exemple simple d'utilisation d'OpenBLAS en C pour effectuer une multiplication de matrices :

``` C
#include <stdio.h>
#include <stdlib.h>
#include <cblas.h>

#define N 3

int main() {
    // Déclaration des matrices
    double A[N*N] = {1.0, 2.0, 3.0,
                     4.0, 5.0, 6.0,
                     7.0, 8.0, 9.0};
    double B[N*N] = {9.0, 8.0, 7.0,
                     6.0, 5.0, 4.0,
                     3.0, 2.0, 1.0};
    double C[N*N] = {0.0};

    // Multiplication des matrices
    cblas_dgemm(CblasRowMajor, CblasNoTrans, CblasNoTrans, N, N, N, 1.0, A, N, B, N, 0.0, C, N);

    // Affichage du résultat
    printf("Résultat de la multiplication de matrices :\n");
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            printf("%f ", C[i*N + j]);
        }
        printf("\n");
    }

    return 0;
}
```
Dans cet exemple :

Nous définissons trois matrices A, B, et C de taille 3x3.
Nous utilisons la fonction **cblas_dgemm** d'OpenBLAS pour effectuer la multiplication des matrices A et B, stockant le résultat dans la matrice C.
Enfin, nous affichons le résultat de la multiplication de matrices.

# Soumission avec Slurm
Maintenant, vous devez créer encore un autre script SLURM **job_matrices.sh** pour soumettre votre travail. Voici un exemple de script SLURM :

``` C
#!/bin/bash
#SBATCH --job-name=job_matrices
#SBATCH --output=job_matrices.out
#SBATCH --error=job_matrices.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=1:00:00

# Chargement du module OpenBLAS
module purge
module load OpenBLAS/0.3.26/gcc-13.1.0

# Exécution du script OpenBLAS
gcc -o matrices matrices.c -lopenblas
./matrices
```

Pour exécuter le code OpenBLAS, soumettez simplement le travail à SLURM avec la commande suivante :
``` C
$ sbatch ./job_matrices.sh

```
