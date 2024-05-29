# HPC - Didacticiel d'utilisation de FORTRAN sur Taouey  

## Comment soumettre un job fortran avec SLURM
Afin de soumettre des tâches FORTRAN à SLURM, nous aurons besoin de deux code sources :

* (1) un script FORTRAN qui décrit la solution à votre problème et,
* (2) un script SLURM qui spécifie les ressources nécessaires, définit l'environnement et les commandes à exécuter.

# Exemple du code Fortran90
Supposons qu’on a un exemple hello.f90 simple pour commencer :

``` FORTRAN
program hello
    implicit none
    print *, "Hello, world!"
end program hello
```

Sauvegardez ce code dans un fichier nommé hello.f90 .

## Compilation du code Fortran 
Pour compiler le code fortran nous pouvons utiliser des compilateurs tels que **gfortran** ou **ifort** sur la plupart des supercalculateurs.


## Écriture du script SLURM :
Maintenant, vous devez créer un script SLURM **hello.sh** pour soumettre votre travail. Voici un exemple de script SLURM :

``` FORTRAN
#!/bin/bash
#SBATCH -J jobFortran             # Nom de la tâche
#SBATCH -N 5                      # Nombre de noeuds a utiliser
#SBATCH -t 00:05:00               # Durée maximale de la tâche
#SBATCH --ntasks=40               # Nombre total de tâhe a exécuter
#SBATCH --output=Res_fort.txt	   # Fichier de sortie

# Compilation du code fortran
gfortran -o hello hello.f90

# Exécution du programme compilé
./hello
```

Pour exécuter le code FORTRAN, soumettez simplement le travail à SLURM avec la commande suivante :
```
$ sbatch ./hello.sh

```
