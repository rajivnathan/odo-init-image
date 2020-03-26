## Notes  

### supervisor.conf file

The /opt/odo/supervisor.conf file calls the `devfile-command` script which in turn runs a command that is set via environment variable.

odo will parse the devfile for commands and create environment variables that will be applied to the containers associated with the commands.

## Demo

kubectl apply -f supervisord.yaml

podname=`kubectl get pods --selector=component=openlibertydevfile --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}'`
kubectl exec $podname -- sh -c "cd /projects/openLiberty && ls && /artifacts/bin/build.sh"

podname=`kubectl get pods --selector=component=openlibertydevfile --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}'`
kubectl logs $podname -f

podname=`kubectl get pods --selector=component=openlibertydevfile --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}'`
kubectl exec $podname -- /opt/odo/bin/supervisord ctl stop all

podname=`kubectl get pods --selector=component=openlibertydevfile --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}'`
kubectl exec $podname -- /opt/odo/bin/supervisord ctl start run

## Questions
### What components should have supervisord?
Only components that have a run command should have supervisord

### Should supervisord always be the entrypoint for `run` components when using odo?
In the initial implementation we'll use supervisord as the entrypoint either via go-init or directly but if the devfile component has a command specified that should be used as the entrypoint instead. supervisord will still be initialized after the container has started and commands will be executed via supervisord. The benefit of executing commands with supervisord is that even if a devfile creator specifies a command that does not run in the background, the odo CLI will still return to the shell.

## Implementation plan
- Only copy supervisord into the container that has a corresponding run command
- If devfile has an entrypoint specified then do not use supervisord as the entrypoint but initialize it after the container has started

## Future considerations
- Provide a way to opt out of using supervisord as the entrypoint by looking for a specific attribute in the devfile
