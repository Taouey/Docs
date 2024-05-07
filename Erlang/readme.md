# HPC - Didacticiel d'utilisation d'Erlang sur Taouey 

## Comment soumettre un job Erlang avec SLURM
Afin de soumettre des tâches OpenCL à SLURM, nous aurons besoin principalement de deux code sources :

* (1) un programme Erlang qui décrit la solution à votre problème et,
* (2) un script SLURM qui spécifie les ressources nécessaires, définit l'environnement et les commandes à exécuter.

# Erlang par l’exemple
Voici une façon d'écrire hello world :
``` C
-module(hello).
	-export([hello_world/0]).

	hello_world() -> io:fwrite("hello, world\n").
```
N'oubliez pas le point (.) à la fin de chaque commande, comme indiqué.

Pour compiler ce texte, enregistrez-le dans un fichier appelé **hello.erl** à compiler à partir de l'interpréteur de commandes **erlang**. 

## Écriture du script SLURM 
Maintenant, vous devez créer un script SLURM **hello_erlang.sh** pour soumettre votre travail. Voici un exemple de script SLURM :

``` C
#!/bin/bash
#SBATCH --job-name=hello_erlang
#SBATCH --output=hello_erlang.out
#SBATCH --error=hello_erlang.err
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --time=1:00:00

# Chargement du module Erlang
module purge
module load Erlang/26.2.5./Erlang-26.2.5.

# Compiler le programme Erlang
erlc hello.erl

# Exécuter le code Erlang 
erl -noshell -s hello hello_world -s init stop
```

Pour exécuter le code Erlang, soumettez simplement le travail à SLURM avec la commande suivante :
```
$ sbatch ./hello_erlang.sh
```

# Suivi des job Erlang sur SLURM
Afin de suivre l'état d'avancement de votre job une fois soumis, se référer au didacticiel sur SLURM.  
cat("[Didacticiel sur SLURM](https://github.com/DiopBabacarEdu/TaoueY-HPC/tree/main/SLURM)")


