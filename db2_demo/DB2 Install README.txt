DB2 Install on RHOS

PREREQS:

	Your local environment Should have:

		a) a git client
		b) https://docs.docker.com/install/linux/docker-ce/ubuntu/
		c) https://cloud.ibm.com/docs/cli?topic=cloud-cli-getting-started
		d) IBM Cloud CLI - https://cloud.ibm.com/docs/container-registry-cli-plugin?topic=container-registry-cli-plugin-containerregcli
		e) Openshift CLI - https://cloud.ibm.com/docs/openshift?topic=openshift-openshift-cli
			(we'll be concentrating here today...)

		1) An openshift project
		2) A SecurityC ontext within the project

Helpful links to get started:

	RHOS CLI Reference

		https://docs.openshift.com/container-platform/3.11/cli_reference/basic_cli_operations.html

	Understandng Security Context in RHOS

		https://developers.redhat.com/blog/2016/10/21/understanding-openshift-security-context-constraints/
		https://docs.openshift.com/container-platform/3.9/admin_guide/manage_scc.html

	Understaning RHOS Compute Measures

		https://docs.openshift.com/enterprise/3.1/dev_guide/compute_resources.html#compute-resource-units


1) https://github.com/IBM/charts/tree/master/stable/ibm-db2oltp-dev

	Login and Clone repo 

	I use a shared folder between VM and host where I keep my repos. For demo, I created seperate directories to keep structures independent and simple.


2) Edit the values.yaml for DB2

	What did I edit?

	- Ports
	- DB Name
	- DB Password
	- dataVolume: storageClassName, size
	- resources: requests: memory, cpu


3) Add policy and adm policy to project

   ##oc policy add-role-to-group system:image-puller system:serviceaccounts:${NAMESPACE} -n ${NAMESPACE}
   ##oc adm policy add-scc-to-user privileged  "system:serviceaccount:${NAMESPACE}:default"

4) Helm install the DB2 StatefulSet

	##helm install --namespace <OS_PROJECTNAME> --tiller-namespace tiller --name <RELEASE_NAME>  -f <RELATIVE PATH_TO /ibm-db2oltp-dev/values.yaml> <RELATIVE PATH TO ibm-db2oltp-dev DIRECTORY>


commands used:

	oc policy add-role-to-group system:image-puller system:serviceaccounts:db2-demo -n db2-demo
	oc adm policy add-scc-to-user privileged  "system:serviceaccount:oms-apawar:default"
	helm install --namespace db2-demo --tiller-namespace tiller --name db2-demo01  -f ./ibm-db2oltp-dev/values.yaml ./ibm-db2oltp-dev

Next Steps:

	/opt/ibm/db2/V11.5/bin/db2 'CONNECT to demodb user db2inst1 using demo'


	Create service if externalization needed either by:

	CLI:
		oc create -f <filename>

	Web UI:
	
		Add to Project	

Template yaml:

apiVersion: v1
kind: Service
metadata:
  name: <SVC_NAME>
spec:
  ports:
  - name: main
    port: <PORT>
  loadBalancerIP:
  type: LoadBalancer
  selector:
    app: <LABEL_APP_NAME>
    component: <LABEL_COMPONENT>

Demo based Example:

apiVersion: v1
kind: Service
metadata:
  name: db2demo-tcp
spec:
  ports:
  - name: main
    port: 50000
  loadBalancerIP:
  type: LoadBalancer
  selector:
    app: db2-demo01-ibm-db2oltp-dev-0
    component: db2