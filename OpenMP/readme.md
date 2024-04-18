# HPC - Didacticiel d'utilisation de OpenMP sur Taouey
* OpenMP est une API pour la programmation parallèle sur architectures à mémoire partagée
* Souvent utilisée pour accélérer les calculs sur des systèmes multi-cœurs. 
## Soumission de tâches OpenMP via SLURM

Afin de soumettre des tâches OpenMP à SLURM, nous aurons besoin de deux code sources : 
* (1) un **script OpenMP** écrit dans le langage de votre choix qui décrit la solution à votre problème et,
* (2) un **script SLURM** qui spécifie les ressources nécessaires, définit l'environnement et les commandes à exécuter.
Dans cet exemple, nous utilisons OpenMP avec la version **gcc-13.1.0**.

## Exécution d'une tâche avec OpenMP
Un job OpenMP est une tâche qui peut nécessiter un ou plusieurs cœurs de CPU. Voici un exemple de script OpenMP (hello_world.c) :

```C
#include <stdio.h>
#include <omp.h>

int main() {
    // Début de la région parallèle
    #pragma omp parallel
    {
        // Obtenez l'identifiant du thread
        int tid = omp_get_thread_num();
        // Obtenez le nombre total de threads
        int num_threads = omp_get_num_threads();
        printf("Hello, world! Je suis le thread %d sur %d.\n", tid, num_threads);
    } // Fin de la région parallèle

    return 0;
}
```
Le script Slurm ci-dessous peut être utilisé pour soumettre un job OpenMP :

```C
#!/bin/bash
#SBATCH --job-name=hello_world_  # Donner un nom à votre job
#SBATCH --nodes=1                # ID du noeud
#SBATCH --ntasks=1               # Allouer le nombre total de tâches
#SBATCH --cpus-per-task=1        # Cpu-cores par tâche 
#SBATCH --mem-per-cpu=4G         # Mémoire allouée par cpu-core 
#SBATCH --time=00:01:00          # Limite de temps d'exéctution (HH:MM:SS)
#SBATCH --mail-type=all          # Notifications en cas de démarrage, d'arrêt et d'erreur

# Chargement du module GCC
module purge
module load gcc/13.1.0/gcc-13.1.0

#Compilation du code
gcc -fopenmp hello_world.c -o hello_world

# Exécution du programme
./hello_world
```

Pour exécuter ce script, soumettez simplement le travail à SLURM avec la commande suivante :
```C
$ sbatch ./hello_world.sh
```
## Suivi d'un job OpenMP 
Une fois la tâche soumise, il est nécessaire de pouvoir suivre l'évolution de celle-ci au fur et à mesure qu'elle s'éxécute sur Taouey.
Voici la commande pour afficher le statut d'une tâche :
```C
squeue -u $USER
```
## Gérer les versions gcc aux travers des modules
Nous avons utilisé dans notre exemple la version 13.1.0 de gcc. Cependant, au cas où il serait nécessaire de se calquer sur d'autres versions, 
taper la commande qui permet de lister les autres versions disponibles.

```
$ module avail mpi
------------ /software/modulefiles ----------------------------
gcc/12.1.0/gcc-12.1.0       gcc/13.1.0/gcc-13.1.0
------------ /etc/modulefiles ---------------------------------

```
Ensuite, charger le module correspondant en tapant la commande suivante
```
module load gcc/12.1.0/gcc-12.1.0
```


## Parallélisation d'un job OpenMP 

Afin de paralléliser un job OpenMP, il est judicieux de procéder à une subdivision de la tâche globale en plusieurs sous-tâches,
lesquelles peuvent être exécutées en parallèle sur plusieurs processeurs. 


Voici un exemple simple de code parallèle en C utilisant OpenMP pour calculer la somme des éléments d'un tableau :
```
#include <stdio.h>
#include <omp.h>

#define ARRAY_SIZE 100

int main() {
    int sum = 0;
    int i;

    // Début de la région parallèle
    #pragma omp parallel for reduction(+:sum)
    for (i = 0; i < ARRAY_SIZE; i++) {
        sum += i; // Exemple: calcul de la somme des indices
    }
    // Fin de la région parallèle

    printf("La somme globale est : %d\n", sum);

    return 0;
}
```
Voici le script SLURM ci-dessous pour soumettre cette tâche :
```C
#!/bin/bash
#SBATCH --job-name=open_mp_job   # Donner un nom à votre job
#SBATCH --nodes=1                # ID du noeud
#SBATCH --ntasks=1               # Allouer le nombre total de tâches
#SBATCH --cpus-per-task=1        # Cpu-cores par tâche 
#SBATCH --mem-per-cpu=4G         # Mémoire allouée par cpu-core 
#SBATCH --time=00:01:00          # Limite de temps d'exéctution (HH:MM:SS)
#SBATCH --mail-type=all          # Notifications en cas de démarrage, d'arrêt et d'erreur

# Chargement du module GCC
module purge
module load gcc/13.1.0/gcc-13.1.0

#Compilation du code
gcc -fopenmp test.c -o test

# Exécution du programme
./test
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



