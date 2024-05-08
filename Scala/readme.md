# HPC - Didacticiel d'utilisation de Scala sur Taouey 

## Comment soumettre un job Scala avec SLURM
Afin de soumettre des tâches Scala à SLURM, nous aurons besoin principalement de deux code sources :

* (1) un programme Scala qui décrit la solution à votre problème et,
* (2) un script SLURM qui spécifie les ressources nécessaires, définit l'environnement et les commandes à exécuter.

# Scala par l’exemple
Voici une façon d'écrire un programme "hello World" :
``` C
object HelloWorld extends App {
  println("Hello world" + args.mkString(" "))
}
```

Pour compiler ce programme, enregistrez-le dans un fichier appelé **hello.scala**.

## Écriture du script SLURM 
Maintenant, vous devez créer un script SLURM **hello_Scala.sh** pour soumettre votre travail. Voici un exemple de script SLURM :

``` C
#!/bin/bash
#SBATCH --job-name=hello_Scala
#SBATCH --output=hello_Scala.out
#SBATCH --error=hello_Scala.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=1:00:00

# Chargement du module Scala
module purge
module load Scala

# Compiler le programme Scala
scalac hello.scala

# Exécuter le code Scala 
scala hello.scala
```

Pour exécuter le code Scala, soumettez simplement le travail à SLURM avec la commande suivante :
```
$ sbatch ./hello_Scala.sh
```

# Suivi des job Scala sur SLURM
Afin de suivre l'état d'avancement de votre job une fois soumis, se référer au didacticiel sur SLURM.  
cat("[Didacticiel sur SLURM](https://github.com/DiopBabacarEdu/TaoueY-HPC/tree/main/SLURM)")


