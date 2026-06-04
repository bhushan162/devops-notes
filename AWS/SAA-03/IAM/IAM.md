# IAM 
- Identity access managment , root account, user and groups 
- IAM is Global Service
- 

## IAM Permission 
- User and group can be asssigne Json Document like below 

`{
    "version": "2026-5-29",
    
    "Action": "ec2:describe",
    "Resource": "*"
}`


#### IAM USER Login
- IAM user login using login URL OR AS IAM USER
-  There is one functnality called multi-session login where we can use two or account on same Brouser 
-  else you have to use inconginito mode for 2nd account 
-  


## IAM Policies 
- this are YAML file that attach to group so group user inhirits that policies 
- plz see the "s3_Account_permission.yaml"

## IAM MFA
- there are some password policies are like length, special caracter ect.

## Good Practices
- Dont use Root user for regular work.
- make one admin user for all admin works 
- 