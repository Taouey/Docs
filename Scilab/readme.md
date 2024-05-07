# HPC - Didacticiel d'utilisation de Scilab sur Taouey 

## Comment soumettre un job Scilab avec SLURM
Afin de soumettre des tâches Scilab à SLURM, nous aurons besoin principalement de deux code sources :

* (1) un script Scilab qui décrit la solution à votre problème et,
* (2) un script SLURM qui spécifie les ressources nécessaires, définit l'environnement et les commandes à exécuter.

# Scilab par l’exemple
Commençons par tester une fonction simple, la fonction plot2d qui trace un ensemble de courbes en **2D**. Étant donné que **x** et **y** sont des vecteurs de même taille, **plot2d(x, y, <opt_args>)** trace le vecteur **y** en fonction du vecteur **x**. 

``` C
// Definir la plage de valeurs de x
x = 0 : .1 : 10*%pi;
// Definir la function
y = x .* cos(x);
// Appel de la fonction plot
plot(x, y)
// Ajouter titres, labels et légendes
title('x * cos(x)')
xlabel('x'); ylabel('y');
legend('y = x * cos(x)');
```
Sauvegardez ce code dans un fichier nommé **test_Scilab.sci**.

## Écriture du script SLURM 
Maintenant, vous devez créer un script SLURM **test_Scilab.sh** pour soumettre votre travail. Voici un exemple de script SLURM :

``` C
#!/bin/bash
#SBATCH --job-name=test_Scilab
#SBATCH --output=test_Scilab.out
#SBATCH --error=test_Scilab.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=1:00:00

# Chargement du module Scilab
module purge
module load Scilab/6.1.1/Scilab-6.1.1

# Exécution du script Scilab
scilab-cli -f your_script.sci
```

Pour exécuter le code R, soumettez simplement le travail à SLURM avec la commande suivante :
```
$ sbatch ./test_Scilab.sh
```

# Suivi des job Scilab sur SLURM
Afin de suivre l'état d'avancement de votre job une fois soumis, se référer au didacticiel sur SLURM.  
cat("[Didacticiel sur SLURM](https://github.com/DiopBabacarEdu/TaoueY-HPC/tree/main/SLURM)")


