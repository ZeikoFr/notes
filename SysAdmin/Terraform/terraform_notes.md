# Variables

## Types de variables

- String
- Number
- Bool
- List
- Map

Si une variable n'est pas définie elle sera demandée à l'application du plan

## Ordre des variables 
(par ordre de force (1 < 5)
1) Environnement
2) fichier : terraform.tfvars
3) fichier json : terraform.tfvars.json
4) fichier *.auto.tfvars
5) CLI -var ou -var-file
