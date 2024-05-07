# HPC - Didacticiel d'utilisation de Mathematica sur Taouey 

## Comment soumettre un job Mathematica avec SLURM
Afin de soumettre des tâches Mathematica à SLURM, nous aurons besoin principalement de deux code sources :

* (1) un script Mathematica qui décrit la solution à votre problème et,
* (2) un script SLURM qui spécifie les ressources nécessaires, définit l'environnement et les commandes à exécuter.

# Mathematica par l’exemple
Voici une façon d'écrire un programme qui calcule les racines d'une équation quadratique :
``` C
(* Define the coefficients of the quadratic equation *)
a = 1;
b = -3;
c = 2;

(* Calculer les racines *)
roots = Solve[a*x^2 + b*x + c == 0, x];

(* Afficher les racines *)
roots
```

Enregistrez le code dans un fichier appelé **hello.m**.

## Écriture du script SLURM 
Maintenant, vous devez créer un script SLURM **hello_mathematica.sh** pour soumettre votre travail. Voici un exemple de script SLURM :

``` C
#!/bin/bash
#SBATCH --job-name=hello_mathematica
#SBATCH --output=hello_mathematica.out
#SBATCH --error=hello_mathematica.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=1:00:00

# Chargement du module Mathematica
module purge
module load Mathematica/13/mathematica-13.0

# Exécuter le code  
math -script hello.m
```

Pour exécuter le code Mathematica, soumettez simplement le travail à SLURM avec la commande suivante :
```
$ sbatch ./hello_mathematica.sh
```

# Suivi des job Mathematica sur SLURM
Afin de suivre l'état d'avancement de votre job une fois soumis, se référer au didacticiel sur SLURM.  
cat("[Didacticiel sur SLURM](https://github.com/DiopBabacarEdu/TaoueY-HPC/tree/main/SLURM)")


