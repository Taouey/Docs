# HPC - Didacticiel d'utilisation de Chapel sur Taouey 

## Comment soumettre un job Chapel avec SLURM
Afin de soumettre des tâches Chapel à SLURM, nous aurons besoin principalement de deux code sources :

* (1) un programme Chapel qui décrit la solution à votre problème et,
* (2) un script SLURM qui spécifie les ressources nécessaires, définit l'environnement et les commandes à exécuter.

# Chapel par l’exemple
Voici une façon d'écrire un programme qui génère un tableau d'entiers de nombres aléatoires et calcule la somme globale :
``` C
// hello.chpl
use Random;

// Generer un tableau de nombres aléatoires
proc generateRandomArray(size: int): [1..size] real {
  var A: [1..size] real;
  forall i in A.domain do
    A[i] = rand(0.0, 100.0);
  return A;
}

// Calculer la somme du tableau
proc sumArray(A: [1..size] real): real {
  var sum: real = 0.0;
  forall i in A do
    sum += A[i];
  return sum;
}

// Fonction main principale
proc main() {
  const size = 10;
  var data = generateRandomArray(size);
  writeln("Generated array:", data);
  var total = sumArray(data);
  writeln("Sum of array:", total);
}

// Commencer éxécution
main();

```

Pour compiler ce programme, enregistrez-le dans un fichier appelé **hello.chpl**.

## Écriture du script SLURM 
Maintenant, vous devez créer un script SLURM **hello_chapel.sh** pour soumettre votre travail. Voici un exemple de script SLURM :

``` C
#!/bin/bash
#SBATCH --job-name=hello_chapel
#SBATCH --output=hello_chapel.out
#SBATCH --error=hello_chapel.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=1:00:00

# Chargement du module Chapel
module purge
module load Chapel/2.0/Chapel-2.0

# Compiler le programme Chapel
chpl hello.chpl -o hello

# Exécuter le code Chapel 
./hello
```

Pour exécuter le code Chapel, soumettez simplement le travail à SLURM avec la commande suivante :
```
$ sbatch ./hello_chapel.sh
```

# Suivi des job Chapel sur SLURM
Afin de suivre l'état d'avancement de votre job une fois soumis, se référer au didacticiel sur SLURM.  
cat("[Didacticiel sur SLURM](https://github.com/DiopBabacarEdu/TaoueY-HPC/tree/main/SLURM)")


