# Helm Chart for iViva Account creation
This script is not part of installation, as we might need to create multiple accounts in later stages.

This script if installed after ocb creation, need to enter those values in `model-settings.json` in a specific format mentioned below


# Procedure
This should be installed in same namespace as iviva

Go to the specific location and install this with command below
```
helm install <accountname> . -n <ivivanamespace>
```
You will need github personal access token for the repo from which you are going to create accounts, get that and keep it handy.

For now, it is 
```
https://github.com/shivanan/mda-account-creation/
```
`https://github.com/shivanan/mda-account-creation/blob/main/model-settings.json` This file requires a value of `ocb url` which should be in the following format

`http://<servicename>.<namespacename>:1026` where `servicename` is the service name of orion which was installed with helm chart, and `namespacename` is the name of namespace in which it is present.

`values.yaml` needs following things which has self-explanatory comments

```
iviva:
  FWid: "f8ce4aa1" ###### FW id from Eutech (should be same as installed one)
  appsid: "4d54331d" ####### apps id from Eutech (should be same as installed one)
  uxpid: "latest"  ###### uxp id from Eutech (should be same as installed one)
  releasename: "ivivac" ### Enter releasename of iviva
  accountname: "accountname" ##Enter the name of the account that need to be created
  accountfirstname: "new" ### Enter the firstname of the account that is being created
  accountlastname: "user" ### Enter the firstname of the account that is being created
  accountusername: "admin" ### Enter the username of the account that is being created
  accountpassword: "password" ### Enter the password of the account that is being created
  accountemailid: "haran@ecyber.com"  ### Enter the emailid of the account that is being created
  gitpersonalaccesstoken: #######Enter personal access token created from github
  giturl: ###Enter URL of repo without https://
```
After creating the account, access it by the url with the credentials given on `values.yaml`.

You need to add this account to keycloak if you want this to be accessed through sso

# Known Issues

There might be some issue due to keycloak, which might take sometime to load. 