# opa_gatekeeper

**What is OPA:**

    The Open Policy Agent (OPA) is an open-source, general-purpose policy engine that unifies policy enforcement across the stack. 
    OPA provides a high-level declarative language that lets us specify policies as code and simple APIs to offload policy decision-making from our software. 
    We can use OPA to enforce policies in microservices, Kubernetes, CI/CD pipelines, API gateways, and more. 
    In kubernetes, OPA uses admission controllers.

**What is OPA Gatekeeper?**

OPA Gatekeeper is a specialized project providing first-class integration between OPA and Kubernetes.

OPA Gatekeeper adds the following on top of plain OPA:

● An extensible, parameterized policy library.

● Native Kubernetes CRDs for instantiating the policy library (aka “constraints”).

● Native Kubernetes CRDs for extending the policy library (aka “constraint templates”).

● Audit functionality.

![image](https://github.com/tushardashpute/opa_gatekeeper/assets/74225291/99c1ee4f-af5d-4f2e-be38-6a665d3bfcfb)

**Gatekeeper Installation:**

    kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/master/deploy/gatekeeper.yaml

Following are the objects created as part of the gatekeeper installation:

    kubectl get all -n gatekeeper-system
    NAME                                                READY   STATUS    RESTARTS        AGE
    pod/gatekeeper-audit-84db645d97-nfn45               1/1     Running   1 (3h22m ago)   3h23m
    pod/gatekeeper-controller-manager-c796b4944-4j42d   1/1     Running   0               3h23m
    pod/gatekeeper-controller-manager-c796b4944-jnnhn   1/1     Running   0               3h23m
    pod/gatekeeper-controller-manager-c796b4944-kvbh4   1/1     Running   0               3h23m
    
    NAME                                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
    service/gatekeeper-webhook-service   ClusterIP   10.100.227.206   <none>        443/TCP   3h23m
    
    NAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/gatekeeper-audit                1/1     1            1           3h23m
    deployment.apps/gatekeeper-controller-manager   3/3     3            3           3h23m
    
    NAME                                                      DESIRED   CURRENT   READY   AGE
    replicaset.apps/gatekeeper-audit-84db645d97               1         1         1       3h23m
    replicaset.apps/gatekeeper-controller-manager-c796b4944   3         3         3       3h23m

**Validating Admission Control**

        Once all the Gatekeeper components have been installed in our cluster, the API server will trigger the Gatekeeper
    admission webhook to process the admission request whenever a resource in the cluster is created, updated, or deleted.
        During the validation process, Gatekeeper acts as a bridge between the API server and OPA. 
    The API server will enforce all policies executed by OPA.

**CustomResourceDefinition**

    The CustomResourceDefinition ( CRD ) API allows us to define custom resources. 
    Defining a CRD object creates a new custom resource with a name and schema that we specify. 
    The Kubernetes API serves and handles the storage of your custom resources.
    
    Gatekeeper uses CustomResourceDefinitions internally and allows us to define 
    ConstraintTemplates and Constraints to enforce policies on Kubernetes resources such as Pods, Deployments, and Jobs.
    
Gatekeeper creates several CRDs during the installation process :

    # kubectl get crd | grep -i gatekeeper
    assign.mutations.gatekeeper.sh                                      2023-10-02T06:15:37Z
    assignimage.mutations.gatekeeper.sh                                 2023-10-02T06:15:37Z
    assignmetadata.mutations.gatekeeper.sh                              2023-10-02T06:15:37Z
    configs.config.gatekeeper.sh                                        2023-10-02T06:15:37Z
    constraintpodstatuses.status.gatekeeper.sh                          2023-10-02T06:15:37Z
    constrainttemplatepodstatuses.status.gatekeeper.sh                  2023-10-02T06:15:37Z
    constrainttemplates.templates.gatekeeper.sh                         2023-10-02T06:15:38Z
    expansiontemplate.expansion.gatekeeper.sh                           2023-10-02T06:15:38Z
    expansiontemplatepodstatuses.status.gatekeeper.sh                   2023-10-02T06:15:38Z
    modifyset.mutations.gatekeeper.sh                                   2023-10-02T06:15:38Z
    mutatorpodstatuses.status.gatekeeper.sh                             2023-10-02T06:15:38Z
    providers.externaldata.gatekeeper.sh                                2023-10-02T06:15:38Z


One of them is “constrainttemplates.templates.gatekeeper.sh” using that 
we can create Constraints and Constraint Templates to work with gatekeeper:

![image](https://github.com/tushardashpute/opa_gatekeeper/assets/74225291/47613d3e-e554-4b83-9fa9-6c4129570b2b)

● [**ConstraintTemplates**](https://open-policy-agent.github.io/gatekeeper/website/docs/howto/#constraint-templates) define a way to validate some set of Kubernetes objects in Gatekeeper’s Kubernetes admission controller. 
They are made of two main elements:

 1. [**Rego**](https://www.openpolicyagent.org/docs/latest/#rego) code that defines a policy violation
 2. The schema of the accompanying Constraint object, which represents an instantiation of a ConstraintTemplate

    ● A Constraint is a declaration of requirements that a system needs to meet. 
    In another word, Constraints are used to inform Gatekeeper that the admin wants a
    ConstraintTemplate to be enforced, and how.

![image](https://github.com/tushardashpute/opa_gatekeeper/assets/74225291/1b9ad277-9530-4adc-adc6-bd7c5a95d626)

Following is an illustration of how CRD, Contraint Template, and Constraint connect with each other:

![image](https://github.com/tushardashpute/opa_gatekeeper/assets/74225291/5c57fc23-3be5-43a0-9330-964145b7c5f6)

**Walkthrough**
Now let’s say we want to enforce a policy so that a kubernetes resource (such as a pod, namespace, etc) must have a particular label defined. 
To achieve that let’s create a ConstraintTemplate first and then create a Constraint :
