# AWS Organizational Unit (OU)
An organizational unit (OU) is a logical grouping of accounts in an organization, created using AWS Organizations. 
OUs enable us to organize our accounts into a hierarchy making it easier to apply management controls. More about OUs [here](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_getting-started_concepts.html).

## How do we use them in the operator ?
We use OUs for billing and reporting, we achieve this by creating an OU to represent a customer entity and placing any accounts (clusters) owned by that entity in the created OU.
The OU name is derived from the [`legalEntity.id`](https://github.com/openshift/aws-account-operator/blob/58923bc2ab9f33d51bc129f6e01a593bcc943b28/deploy/crds/aws.managed.openshift.io_accountclaims.yaml#L107) field in the `AccountClaim`, 
we chose to use the `id` rather than `name` to guarantee uniqueness. We also leverage the operator [`Configmap`](https://github.com/openshift/aws-account-operator/blob/58923bc2ab9f33d51bc129f6e01a593bcc943b28/hack/templates/aws.managed.openshift.io_v1alpha1_configmap.tmpl)
to store the `ID's` of the [root OU](https://github.com/openshift/aws-account-operator/blob/58923bc2ab9f33d51bc129f6e01a593bcc943b28/hack/templates/aws.managed.openshift.io_v1alpha1_configmap.tmpl#L20), 
which is where all AWS `Accounts` will be located by default, and the [base OU](https://github.com/openshift/aws-account-operator/blob/58923bc2ab9f33d51bc129f6e01a593bcc943b28/hack/templates/aws.managed.openshift.io_v1alpha1_configmap.tmpl#L21) 
that maps the location we want to create other OUs under.

## Where does it happen?
The entire logic is located within the [`AccountClaim`](https://github.com/openshift/aws-account-operator/tree/58923bc2ab9f33d51bc129f6e01a593bcc943b28/pkg/controller/accountclaim) controller, in particular the [`MoveAccountToOU`](https://github.com/openshift/aws-account-operator/blob/58923bc2ab9f33d51bc129f6e01a593bcc943b28/pkg/controller/accountclaim/organizational_units.go#L19) 
function which does the following:

* Create AWS Client
* Ensure that the operator configmap is present
* Retrieve and validate base and root OU ID values
* Create a new OU or find an existing one for the given entity
* Move account into OU
* Update the [`AccountClaim.AccountOU`](https://github.com/openshift/aws-account-operator/blob/58923bc2ab9f33d51bc129f6e01a593bcc943b28/pkg/apis/aws/v1alpha1/accountclaim_types.go#L20) field with the OU ID.