## Kubernetes Pod Design for Application Developers: Labels, Selectors, and Annotations

#### 1. Create a Namespace for the resources you will create in this lab step and change your default kubectl context to use the Namespace:

#### Create namespace
`kubectl create namespace labels`

#### Set namespace as the default for the current context
`kubectl config set-context $(kubectl config current-context) --namespace=labels`
 > It is a best practice to use namespaces to logically organize your Kubernetes resources. Namespaces identify resources more coarsely than labels and complement labels.

#### 2. Create four Pods declared in the following multi-resource manifest file (pod-labels.yaml):
#### Copy code and write the manifest file
```
cat << 'EOF' > pod-labels.yaml
apiVersion: v1
kind: Pod
metadata:
  name: red-frontend
  namespace: labels # declare namespace in metadata 
  labels: # labels mapping in metadata
    color: red
    tier: frontend
  annotations: # Example annotation
    Lab: Kubernetes Pod Design for Application Developers
spec:
  containers:
  - image: httpd:2.4.38
    name: web-server
---
apiVersion: v1
kind: Pod
metadata:
  name: green-frontend
  namespace: labels
  labels:
    color: green
    tier: frontend
spec:
  containers:
  - image: httpd:2.4.38
    name: web-server
---
apiVersion: v1
kind: Pod
metadata:
  name: red-backend
  namespace: labels
  labels:
    color: red
    tier: backend
spec:
  containers:
  - image: postgres:11.2-alpine
    name: db
---
apiVersion: v1
kind: Pod
metadata:
  name: blue-backend
  namespace: labels
  labels:
    color: blue
    tier: backend
spec:
  containers:
  - image: postgres:11.2-alpine
    name: db
EOF
```
#### Create the Pods

```kubectl create -f pod-labels.yaml```
>Note that multiple resources are separated by --- in YAML manifest files. Each pod is declared in the labels namespace. Each Pod also has a labels mapping that declares a color and a tier label. Values for the color label span red, green, and blue. Values for the tier label span frontend and backend. The first Pod in the file, named red-frontend, also declares an annotations mapping.

 

#### 3. Use the -L (or --label-columns) kubectl get option to display columns for both labels:
`kubectl get pods -L color,tier`

>The following instructions use label selectors to select a set of the Pods to display using kubectl. In cases where you don't know what labels are set on Pods, you can use --show-labels option to display all labels for each pod under a single labels column.

 #### 4. Use the -l (or --selector) option to select all Pods with a color label:

`kubectl get pods -L color,tier -l color`
>Specifying only a label key as a selector will select all resources with that label defined. In this case, all Pods have a color label, so they are all selected for output.

#### 5. Select all Pods that do not have a color label:

`kubectl get pods -L color,tier -l '!color'`

> You can prepend an exclamation mark (!) to select resources without a label defined. The single quotes (') must enclose the selector to prevent the bash shell from interpreting the exclamation mark. You can always enclose your selectors in single quotes to avoid any unexpected consequences of the shell interpreter.

 
#### 6. Select all Pods that have the color red:

`kubectl get pods -L color,tier -l 'color=red'`

>You can select a key and value by joining the key and value with an equal sign (=) or two equal signs (==).

#### 7. Select all Pods that have the color red and are not in frontend tier:

`kubectl get pods -L color,tier -l 'color=red,tier!=frontend'`

>First, note that multiple conditions are joined by commas (,). Second, the != symbol means not equals. For not equals conditions, the given label key must exist. For example, Pods without any tier label would not be selected by the command above.

#### 8. Select all Pods with green or blue color:

`kubectl get pods -L color,tier -l 'color in (blue,green)'`

> The in condition allows you to specify the allowed values in parentheses. There is also a notin condition that allows you to specify disallowed values.

#### 9. Use the describe command to display the annotation for the red-frontend Pod:

`kubectl describe pod red-frontend | grep Annotations`

>The describe command is a convenient way to display annotations. You can also set the output option (-o) of get to yaml to view all the fields of resources, including annotations, e.g. kubectl get pod red-frontend -o yaml.

#### 10. Remove the Pod's Lab annotation and verify it no longer exists:
`
kubectl annotate pod red-frontend Lab- && kubectl describe pod red-frontend | grep Annotations -A 3
`
>Only annotations related to the cluster’s network plugin remain. The annotate command can be used to add/remove/update annotations. You add a dash (-) after the annotation key to remove the annotation. You can do the same with the kubectl label command when you need to remove a label.

#### 11. Review the Examples in the annotate help:

`kubectl annotate --help`

#### 12. Review the Examples in the label help:

`kubectl label --help'

> The annotate and label commands are analogous. As the differences in the examples highlight, annotations have less restrictions on their values. For example, label values cannot have spaces. 

#### 13. Delete the Pods:

`kubectl delete -f pod-labels.yaml`

#### Summary 
In this lab step, you learned the differences between labels and annotations in Kubernetes. You also learned how selectors are used to select sets of resources by writing label conditions. Selectors are helpful for filtering output using kubectl.

---
#### TIP
<b>`kubectl config set-context $(kubectl config current-context) --namespace=labels`</b>

##### Wyjaśnienie

`kubectl config current-context`
>To polecenie zwraca nazwę bieżącego kontekstu, który jest aktualnie używany przez kubectl. Kontekst zawiera informacje takie jak klaster, użytkownik i przestrzeń nazw, które są używane domyślnie przy wykonywaniu poleceń kubectl.

`$(kubectl config current-context)`
>Użycie $() powoduje, że wynik polecenia kubectl config current-context jest wstawiany w to miejsce. W efekcie to polecenie zostanie zastąpione nazwą bieżącego kontekstu.

`kubectl config set-context <current-context> --namespace=labels`
>To polecenie ustawia przestrzeń nazw labels jako domyślną dla podanego kontekstu (w tym przypadku bieżącego kontekstu). Oznacza to, że wszystkie przyszłe polecenia kubectl będą domyślnie wykonywane w przestrzeni nazw labels, chyba że zostanie to nadpisane przez opcję --namespace w konkretnym poleceniu.

> <b>Podsumowując, pełne polecenie 
`kubectl config set-context $(kubectl config current-context) --namespace=labels` 
ustawia przestrzeń nazw labels jako domyślną dla bieżącego kontekstu kubectl. Po wykonaniu tego polecenia, wszystkie operacje kubectl będą domyślnie wykonywane w przestrzeni nazw labels.</b>