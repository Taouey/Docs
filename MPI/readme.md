# HPC - Didacticiel d'utilisation de MPI sur Taouey

## Soumission de tâches MPI via SLURM

Afin de soumettre des tâches MPI à SLURM, nous aurons besoin de deux code sources : 
* (1) un **script MPI** qui décrit la solution à votre problème et,
* (2) un **script SLURM** qui spécifie les ressources nécessaires, définit l'environnement et les commandes à exécuter.

Dans un premier temps, commençons d'abord par voir comment paralléliser un travail de calcul simple en distribuant le calcul sur plusieurs noeuds.

## Parallélisation d'un job MPI 

Afin de paralléliser un job MPI, il est judicieux de procéder à une subdivision de la tâche globale en plusieurs sous-tâches,
lesquelles peuvent être exécutées en parallèle sur plusieurs processeurs. 

### Quelques conseils à la parallélisation :

Nous posons ici le problème du **calcul la somme des éléments d'un tableau**. Voici quelques étapes qui pourrait aider à paralléliser ce problème sur MPI :

* **Diviser les données** : Diviser le tableau en parties égales (ou aussi équitablement que possible) pour que chaque processus MPI puisse effectuer des calculs sur une partie des données.
* **Distribuer les données** : Utiliser des communications MPI pour distribuer les parties du tableau à chaque processus MPI. Pour cela, nous pouvons utiliser des fonctions MPI comme ```MPI_Scatter``` ou ```MPI_Send/MPI_Recv``` pour cela.
* **Calculer localement** : Chaque processus MPI calcule localement la somme des éléments dans sa partie du tableau.
* **Collecter les résultats** : Utiliser des communications MPI pour collecter les résultats partiels de chaque processus MPI et les agréger en une somme globale. Pour cela, nous pouvons utiliser des fonctions MPI comme ```MPI_Reduce``` ou ```MPI_Send```/```MPI_Recv``` pour cela.
  
Voici une implémentation de cet exemple :
```
#include <stdio.h>
#include <mpi.h>

#define ARRAY_SIZE 10000

int main(int argc, char** argv) {
    int rank, size;
    int local_sum = 0;
    int global_sum = 0;

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    // Diviser les données (tableau) équitablement
    int chunk_size = ARRAY_SIZE / size;
    int start_index = rank * chunk_size;
    int end_index = start_index + chunk_size;

    // Calculer localement la somme des éléments du tableau
    for (int i = start_index; i < end_index; ++i) {
        local_sum += i; // Exemple: calcul de la somme des indices
    }

    // Collecter les résultats partiels de chaque processus
    MPI_Reduce(&local_sum, &global_sum, 1, MPI_INT, MPI_SUM, 0, MPI_COMM_WORLD);

    // Afficher le résultat global sur le processus 0
    if (rank == 0) {
        printf("La somme globale est : %d\n", global_sum);
    }

    MPI_Finalize();
    return 0;
}
```

Afin d'implémenter un job parallèle sur MPI, il faut prévoir une configuration SLURLM avec :
* au minimum deux noeuds alloués,
* et au minimum une tâche pour chaque noeud alloué.

Voici le script SLURM ci-dessous pour soumettre cette tâche :
```C
#!/bin/bash
#SBATCH --job-name=hello_mpi        # Nom du job
#SBATCH --partition=SKL             # Nombre de la partition (sur Taouey SKL ou KNL)
#SBATCH --nodes=5                   # Nombre de noeuds alloués
#SBATCH --ntasks=10                 # Nombre de tâches par noeud 
#SBATCH --time=00:30:00             # Limite de temps d'exéctution (HH:MM:SS)
#SBATCH --output=hello_output.txt   # Nom du fichier de sortie
#SBATCH --err=hello_log.log         # Nom du fichier des logs
#SBATCH --exclusive

module purge
module load mpi/openmpi/4.1.6/gcc-13.1.0

mpicc parallele_mpi.c -o parallele_mpi
mpirun -np 5 ./parallele_mpi
```
Dans cet exemple, nous avons choisi d'allouer 5 noeuds avec pour chacun une seule tâche à exécuter.

Pour exécuter le script MPI, soumettez simplement le travail à SLURM avec la commande suivante :
```C
$ sbatch ./job.sh
```
## Suivi d'un job MPI 
Une fois la tâche soumise, il est nécessaire de pouvoir suivre l'évolution de celle-ci au fur et à mesure qu'elle s'éxécute sur Taouey.
Voici la commande pour afficher le statut d'une tâche :
```C
squeue -u $USER
```
## Gérer les versions aux travers des modules
Nous avons supposé utiliser la version 4.1.6 de MPI. Cependant, au cas où il serait nécessaire de se calquer sur d'autres versions, 
taper la commande qui permet de lister les autres versions disponibles.

```
$ module avail mpi
------------ /software/modulefiles ----------------------------
mpi/openmpi/2.0.4/icc-2018.5.274    mpi/openmpi/3.1.0/gcc-7.2.0
mpi/openmpi/3.1.0/icc-2018.5.274    mpi/openmpi/3.1.6/gcc-4.8.5
------------ /etc/modulefiles ---------------------------------
mpi/mvapich23-x86_64         
```
Ensuite, charger le module correspondant en tapant la commande suivante
```
module load mpi/openmpi/3.1.0/gcc-7.2.0
```

Pour obtenir le nom du nœud de calcul où s'exécute votre travail, utilisez la commande suivante :

```
$ squeue -u $USER
```
La colonne la plus à droite intitulée ```"NODELIST(REASON)"``` donne le nom du nœud où s'exécute votre travail. Connectez-vous via ```SSH``` à ce nœud, par exemple :

```
$ ssh pm1-nod010
```
Une fois sur le nœud de calcul, exécutez ```htop -u $USER```. 
Si votre travail s'exécute en parallèle, vous devriez voir un processus utilisant bien plus de 100 % dans la colonne %CPU. 
















