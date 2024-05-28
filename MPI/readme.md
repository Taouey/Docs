# HPC - Didacticiel d'utilisation de MPI sur Taouey

## Soumission de tâches MPI via SLURM

Afin de soumettre des tâches MPI à SLURM, nous aurons besoin de deux code sources : 
* (1) un **script MPI** qui décrit la solution à votre problème et,
* (2) un **script SLURM** qui spécifie les ressources nécessaires, définit l'environnement et les commandes à exécuter.

## Exécution d'une tâche séquentielle avec MPI
Pour certains cas, l'exécution de MPI en mode single-thread peut répondre à vos besoins. 
Un job MPI séquentiel est une tâche qui peut nécessiter un ou plusieurs cœurs de CPU. 
Voici un exemple de script MPI (hello_world.c) :

```C
#include <mpi.h>
#include <stdio.h>

int main(int argc, char** argv) {
    // Initialiser MPI
    MPI_Init(&argc, &argv);

    // Obtenir le nombre total de processus
    int world_size;
    MPI_Comm_size(MPI_COMM_WORLD, &world_size);

    // Obtenir le rang de ce processus
    int world_rank;
    MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

    // Afficher "Hello, world!" sur chaque processus
    printf("Hello, world! from process %d of %d\n", world_rank, world_size);

    // Finaliser MPI
    MPI_Finalize();
    return 0;
}
```
Le script Slurm ci-dessous peut être utilisé pour des travaux série :

```C
#!/bin/bash
#SBATCH --job-name=hello            # Donner un nom à votre job
#SBATCH --nodes=5                   # Nombre de noeuds alloués
#SBATCH --ntasks=10                 # Nombre total de tâches
#SBATCH --time=00:05:00             # Limite de temps d'exéctution (HH:MM:SS)
#SBATCH --output=hello_output.txt   # Nom du fichier de sortie

module purge
module load mpi/openmpi/4.1.6/gcc-13.1.0

mpicc hello.c -o hello
mpirun -np 4 ./hello
```

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
Nous avons supposé utilisée la version 3.1.6 de MPI. Cependant, au cas où il serait nécessaire de se calquer sur d'autres versions, 
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


## Parallélisation d'un job MPI 

Afin de paralléliser un job MPI, il est judicieux de procéder à une subdivision de la tâche globale en plusieurs sous-tâches,
lesquelles peuvent être exécutées en parallèle sur plusieurs processeurs. 

### Quelques conseils à la parallélisation :

Nous posons ici le problème du **calcul la somme des éléments d'un tableau**. Voici quelques étapes qui pourrait aider à paralléliser ce problème sur MPI :

* **Diviser les données** : Divisez le tableau en parties égales (ou aussi équitablement que possible) pour que chaque processus MPI puisse effectuer des calculs sur une partie des données.
* **Distribuer les données** : Utilisez des communications MPI pour distribuer les parties du tableau à chaque processus MPI. Vous pouvez utiliser des fonctions MPI comme MPI_Scatter ou MPI_Send/MPI_Recv pour cela.
* **Calculer localement** : Chaque processus MPI calcule localement la somme des éléments de sa partie du tableau.
* **Collecter les résultats** : Utilisez des communications MPI pour collecter les résultats partiels de chaque processus MPI et les agréger en une somme globale. Vous pouvez utiliser des fonctions MPI comme MPI_Reduce ou MPI_Send/MPI_Recv pour cela.

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
Voici le script SLURM ci-dessous pour soumettre cette tâche :
```C
#!/bin/bash
#SBATCH --job-name=mpi_job       # Donner un nom à votre job
#SBATCH --nodes=5                # ID du noeud
#SBATCH --ntasks=10              # Allouer le nombre total de tâches
#SBATCH --mail-type=all          # Notifications en cas de démarrage, d'arrêt et d'erreur

module purge
module load mpi/openmpi/4.1.6/gcc-13.1.0

mpicc parallele_mpi.c -o parallele_mpi
mpirun -np 10 ./parallele_mpi
```

Pour obtenir le nom du nœud de calcul où s'exécute votre travail, utilisez la commande suivante :

```
$ squeue -u $USER
```
La colonne la plus à droite intitulée ```"NODELIST(REASON)"``` donne le nom du nœud où s'exécute votre travail. Connectez-vous via SSH à ce nœud, par exemple :

```
$ ssh pm1-nod010
```
Une fois sur le nœud de calcul, exécutez ```htop -u $USER```. 
Si votre travail s'exécute en parallèle, vous devriez voir un processus utilisant bien plus de 100 % dans la colonne %CPU. 
















