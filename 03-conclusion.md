# Conclusion

Why do we need namespaces?
* Create separated environments in one cluster

Why namespaces are good for us?
* In one cluster you can make several small separated environment
* Almost no effect on each other

Why do we need RBAC?
* You can specify roles for user/group
* Limit access to resources

Does it work in production environment?
* Yes

Does it work on managed clusters like AWS,Azure?
* Yes

Is it the best solution?
* No, In NN we use a similar solution. Authentication is made by Aws/Azure. Roles are created in the same way. Account information is loaded with a cron-job into a configmap. Open Policy Agent is an extra tool, responsible to force some validation and modification. Gives more customization and "out of the box" solution.