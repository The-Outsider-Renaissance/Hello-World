
# Entities

## Creating a new entity

**There are three major ways to create new entities:**

> 1.  Manually registering a component.
> 2.  Creating new components through Portal's interface.
> 3.  Integrates with an external source.

The first one applies to all Portal resources and can be provided as a YAML metadata configuration file.

The other two options depend upon [Templates](#templates) or [Plugins](#plugins) to automatically parse information from other sources and generate the resulting files. These automatically provisioned resources will complement the ones provided manually, as most of an entity's definitions are metadata information.

Regardless of whether the entities were generated manually or from other external resources, it is important to understand the main fields that compose each kind of entity.

<sub> [Read more about it: Official Software Catalog section of the external Backstage documentation](https://backstage.io/docs/features/software-catalog/software-catalog-overview).</sub>

!!! Important
    Most objects in Backstage will be composed of YAML strings.
    These strings are to be defined in the form of: `[<kind>:][<namespace>/]<name>` layout.

    They **must** composed of one to three different parts and without any encoding, and that includes quotes when used to reference strings that might contain spaces in another syntax language.

    | Example `owner` spec syntax     |                                      |
    | :---------- | :----------------------------------- |
    | `owner: 'example-team-name'` | :heavy_multiplication_x: Incorrect  |
    | `owner: "example-team-name"` | :heavy_multiplication_x: Incorrect |
    | `owner: example-team-name`   | :material-check-bold: Correct |


### Common specs

There are a few fields that are common to all entities' objects metadata fields. These fields are comprised of:

- `spec.apiVersion` (Required): This value won't change as long the Portal versioning doesn't change, and it must be universal to all Portal objects defined.
- `spec.kind` (Required): Selects the `kind` that entity is a part of. It must be provided and should only be of a single kind.
- `spec.metadata` (Required): Describes additional information about the Portal objects, it'll be at the very least be composed by its name, and it may also have the following additional fields, all of which are optional (**excl. production environments**):

---

### Optional metadata tags

???+ Info

    - `spec.metadata.namespace`: If it is an object that is a part of a specific namespace, these fields are used to define the namespace's name as a string. If it is an entity related to a review environment, this value shouldn't be defined. **This field is required when provisioning resources for staging, pre-production, and production environments.**
    
    - `spec.metadata.title`: A string used to replace the `spec.metadata.name` field on graphical interfaces to increase user-friendliness.
    
    - `spec.metadata.description`: Used to briefly explain to what that resource is used for and its nature.
    
    - `spec.metadata.labels`: Their use is very similar to a Kubernetes label, and is mainly used to reference related entities or classify information.
    
    - `spec.metadata.annotations`: It has identical usage as its Kubernetes counterpart. It is mainly used to reference external systems of any kind and associate them with the entity being defined. It should not describe the component's hierarchy in terms of product structure, please refer to `spec.dependsOn` for specifying non-Kubernetes objects requirements.
    
    - `spec.metadata.tags`: This field has no overall semantics associated with its usage and serves mainly to categorize and help organize resources within Portal's software catalog system. It should be used when you want to list resources per Kind type or driver instead of team or service associated with it.
    
    - `spec.metadata.links`: Links are literal in a sense it must start with an `http://` or `https://` and reference internal or external resources directly, however, the goal of this specific `spec.metadata` is to not to provide any kind of instructions to Backstage - that should be left to Kubernetes-like resources such as `annotations` or `labels`.

---

## Portal entities categories
### Components

``` yaml title="example_component.yaml" linenums="1"
apiVersion: backstage.io/v1alpha1
kind: Component
metadata:
  name: example-name
  description: The place to be, for great artists
spec:
  type: service
  lifecycle: production # (1)
  owner: example-team # (2)
  system: example-system
  dependsOn:
    - resource:default/example-db #(5)
  providesApis:
    - example-api # (3)  (4)
```

1. - **Required**: It defines the component's stage in terms of the development process. **It exclusively has the following strings as field values: `experimental`, `production`, and `deprecated`.**
2. - **Required**: Links a responsible Team or User entity for that specific component. **The values on this field have to follow the guidelines defined by `spec.alifecycle`.**
3. - Optional: An array of entities that references APIs provided to the component being defined.
4. - Optional: An array of entities that references APIs consumed by the component being defined.
5. - Optional: This field value describes how every kind of entity is related to that component on Portal's ecosystem, not only [Components](#components). It mainly differs from `spec.subComponentOf` since it is SRE's and Product Owner's responsibility to update this field and shouldn't be updated by the development team without additional reviews from an infrastructure-related team.


!!! pied-piper "Specs:"

    These are the fields that should be changed when creating a new component definition:

    **1. `spec.lifecycle` (Required)**: It defines the component's stage in terms of the development process.

    **It exclusively has the following strings as field values: `experimental`, `production`, and `deprecated`.**

      - **`experimental` field value** should be applied to services new services that do not have any internal or external users, there's no standard in terms of reliability and it may be subject to unknown periods of downtime.
      - **`production` field value** means that entity is currently being used by users of any kind, and the services with this tag are required to have a team entity assigned to them. The team assigned to the entity's tag are responsible for monitoring, enforcing best practices, and defining those services metadata Portal files.
      - **`deprecated` field value** means the entity with this tag may have both internal and external users within Power, however, regardless of the team's entity responsible for these objects, all `deprecated` tagged entities will be subject to SRE monitoring and review when modifying Portal related files. This prevents unnecessary changes to stable systems and shares the responsibility of knowledge within developers and SREs on the basics of each service, extending support lifetime for them.

    **2. `spec.owner` (Required)**: Links a responsible Team or User entity for that specific component. **The values on this field have to follow the guidelines defined by `spec.lifecycle`.**
    `spec.subcomponentOf` (Optional): It links how components are related and help intuitively define systems hierarchies. This field's value is defined by how developers organize themselves under each service provided by Power infrastructure. The team's responsibilities will be directly linked to this field's value, as the number of components that depend on a certain entity grows, so does the responsibility of the team who owns that entity.

    **3. `spec.providesApis`** (Optional): An array of entities that references APIs provided to the component being defined.

    **4. `spec.consumesApis`** (Optional): An array of entities that references APIs consumed by the component being defined.

    **5. `spec.dependsOn`** (Optional): This field value describes how every kind of entity is related to that component on Portal's ecosystem, not only [Components](#components). It mainly differs from `spec.subComponentOf` since it is SRE's and Product Owner's responsibility to update this field and shouldn't be updated by the development team without additional reviews from an infrastructure-related team.


---

### APIs

  ```yaml title="example_api.yaml" linenums="1"
  apiVersion: backstage.io/v1alpha1
  kind: API
  metadata:
    name: example-api
    description: An example description of what this Portal API does
  spec:
    type: openapi # (1)
    lifecycle: production # (2)
    owner: example-team-name # (3)
    system: example-system-name # (5)
    definition: |   # (4)
      openapi: "3.0.0"
      info:
        version: 1.0.0
        title: Example API
        license:
          name: MIT
      servers:
        - url: http://portal.internal/v1
      paths:
        /example-endpoint:
          get:
            summary: List all endpoints
    ...
  ```

  1.  **Required**: This specification field can assume four different known values:
      - **`openapi`**: Refers to a YAML or JSON format based on [OpenAPI](https://swagger.io/specification/) v2 or v3 specs.
      - **`asyncapi`**: An API definition based on the [AsyncAPI spec](https://www.asyncapi.com/docs/specifications/latest/).
      - **`graphql`**: An API definition based on GraphQL schemas for consuming [GraphQL based APIs](https://graphql.org/).
      - **`grpc`**: An API definition based on [Protocol Buffers](https://developers.google.com/protocol-buffers) to use with [gRPC](https://grpc.io/).
  2.  **Required**: Identical to the component's `spec.lifecycle guidelines for the most part. The only difference is that for `spec.lifecycle: deprecated` value, since it refers exclusively to OpenAPI's definitions. **It is not SRE's responsibility to update and review this entity's specifications**.
  3.  **Required**: Similar to `Component`, the entity `User` or `Group` has to follow the guidelines on the `spec.lifecycle` of API entities. The difference is that APIs cannot be owned by an infrastructure team, however, they can be owned by an individual `User` kind member of an infrastructure team.
  4.  **Required** A string created by using a YAML's [literal style block scalar syntax](https://yaml.org/spec/1.2-old/spec.html#id2795688) with the provided API specifications.
  5. Optional: An array of entity references to the APIs that are consumed by the component, e.g. artist-API. This field is optional.


!!! pied-piper "Specs:"

    **1. `spec.type` (Required)**: This specification field can assume four different known values.
        - **`openapi`**: Refers to a YAML or JSON format based on [OpenAPI](https://swagger.io/specification/) v2 or v3 specs.
        - **`asyncapi`**: An API definition based on the [AsyncAPI spec](https://www.asyncapi.com/docs/specifications/latest/).
        - **`graphql`**: An API definition based on GraphQL schemas for consuming [GraphQL based APIs](https://graphql.org/).
        - **`grpc`**: An API definition based on [Protocol Buffers](https://developers.google.com/protocol-buffers) to use with [gRPC](https://grpc.io/).

    **2. `spec.lifecycle` (Required)**: Identical to the component's `spec.lifecycle guidelines for the most part. The only difference is that for `spec.lifecycle: deprecated` value, since it refers exclusively to OpenAPI's definitions, it is not SRE's responsibility to update and review this entity's specifications.

    **3.`spec.owner` (Required)**: Similar to `Component`, the entity `User` or `Group` has to follow the guidelines on the `spec.lifecycle` of API entities. The difference is that APIs cannot be owned by an infrastructure team, however, they can be owned by an individual `User` kind member of an infrastructure team.

    **4. `spec.definition` (Required)**: A string created by using a YAML's [literal style block scalar syntax](https://yaml.org/spec/1.2-old/spec.html#id2795688) with the provided API specifications.

    **5.`spec.system` (Optional)** : An array of entity references to the APIs that are consumed by the component, e.g. artist-API. This field is optional.


---

### Resources

```yaml title="example_resource.yaml" linenums="1"
apiVersion: backstage.io/v1alpha1
kind: Resource
metadata:
  name: example-db
  description: Example database description
spec:
  type: database # (2)
  owner: example-team # (1)
  system: example-system # (4)
  dependsOn:
    - resource:default/example-db # (3)
```

1. **Required:** Refers to a Portal entity of kind `User` or `Group` responsible for this resource ultimately. As only one entity can be provided, it must be users or groups able to manage these resources using administrative permissions. It is mainly used to map and assign responsibility for the resources provisioning and maintenance.
2. **Required:** This spec field value defines the resource category. These values are not enforced by the spec schemas, but Portal entities should fall in one of these types:
    - `spec.type: database`
    - `spec.type: s3-bucket`
    - `spec.type: cluster`
    - `spec.type: redis`
3. Optional: Refers to other resources that this entity depends upon. This creates the responsibility networking used to inform on what teams have to be notified when a certain resource is affected or is undergoing processes such as a major upgrade.
4. Optional: Refers to the major umbrella application that aggregates together multiple entities. For example Power's Nitro, Milano.

!!! pied-piper "Specs:"

    - **`spec.owner` (Required)**: Refers to a Portal entity of kind `User` or `Group` responsible for this resource ultimately. As only one entity can be provided, it must be users or groups able to manage these resources using administrative permissions. It is mainly used to map and assign responsibility for the resources provisioning and maintenance.
    - **`spec.type` (Required)**: This spec field value defines the resource category. These values are not enforced by the spec schemas, but Portal entities should fall in one of these types:
        - `spec.type: database`
        - `spec.type: s3-bucket`
        - `spec.type: cluster`
        - `spec.type: redis`
    - **`spec.dependsOn`** (Optional): Refers to other resources that this entity depends upon. This creates the responsibility networking used to inform on what teams have to be notified when a certain resource is affected or is undergoing processes such as a major upgrade.
    - **`spec.system`** (Optional): Refers to the major umbrella application that aggregates together multiple entities. For example Power's Nitro, Milano.

---

### Templates

<sub>(i.e Scaffolder plugins, Software Templates)</sub>

[![](./assets/thumb-media.png)](https://backstage.io/blog/assets/2020-08-05/feature.mp4)


[Open and edit a sample entity template on portal :material-circle-edit-outline:](#){ .md-button .md-button--primary }

---

Templates auto-generate additional entities based on a few parameters provided through the Portal interface. It also provided the steps to provide a specific entity's coupled resources

A template has to replicate a repeatable and useful pattern to be useful. At Portal, you can find the available templates [here]().

!!! Info

    ### Available templates

    - [Rails application](https://github.com/backstage/backstage/tree/master/plugins/scaffolder-backend-module-rails)
    - [React frontend](https://github.com/backstage/software-templates/blob/main/scaffolder-templates/create-react-app/template.yaml)

The available templates provide an easy way to create new projects and provide resources to these entities. Their specs can be used to understand how templates are created and be used to expand the current catalog of templates. However, templates depend on Scaffolder plugins to be able to provision resources.

To understand how templates and Scaffolders interact, it is required to read the plugin's documentation:

- [Scaffolder front-end plugin](https://github.com/backstage/backstage/tree/master/plugins/scaffolder)
- [Scaffolder backend plugin](https://github.com/backstage/backstage/tree/master/plugins/scaffolder-backend)
- [Scaffolder backend Rails module plugin](https://github.com/backstage/backstage/tree/master/plugins/scaffolder-backend-module-rails)
- [Scaffolder cookiecutter plugin](https://github.com/backstage/backstage/tree/master/plugins/scaffolder-backend-module-cookiecutter)


```yaml title="example_template.yaml" linenums="1"
apiVersion: backstage.io/v1beta2
kind: Template
# some metadata about the template itself
metadata:
  name: v1beta2-demo
  title: Test Action template
  description: scaffolder v1beta2 template demo
spec:
  owner: backstage/techdocs-core # (3)
  type: service # (1)

  # these are the steps which are rendered in the frontend with the form input
  parameters: # (2)
    - title: Fill in some steps
      required:
        - name
      properties:
        name:
          title: Name
          type: string
          description: Unique name of the component
          ui:autofocus: true
          ui:options:
            rows: 5
    - title: Choose a location
      required:
        - repoUrl
      properties:
        repoUrl:
          title: Repository Location
          type: string
          ui:field: RepoUrlPicker
          ui:options:
            allowedHosts:
              - github.com

(...)
  step:
    - id: register # (2)
      name: Register
      action: catalog:register
      input:
        repoContentsUrl: '{{ steps.publish.output.repoContentsUrl }}'
        catalogInfoPath: '/catalog-info.yaml'
```

  1. **Required**: The entity's resulting `spec.type` being create through this template has to match the template's type property
  2. **Required**: Each `parameter` is a variable that has to be filled by the user of the template when generating a new entity. A parameter can be a `Step`or a list of multiple `Steps`. A `Step` is a [JSONSchema](https://github.com/rjsf-team/react-jsonschema-form) with styling options, you can find further documentation on how `JSONSchema` works in their [official documentation](https://react-jsonschema-form.readthedocs.io/) however, it can also be a `uiSchema`. Both have more examples available [here](https://rjsf-team.github.io/react-jsonschema-form) and [here](https://backstage.io/docs/features/software-templates/writing-templates).
  3. **Required:** A template should be owned by an entity `User` or `Group` responsible for maintaining Portal's overall infrastructure. Teams themselves can submit new templates to be reviewed and approved, but ultimately it should be owned by a Portal's core team.



!!! pied-piper "Specs:"

    Apart from common specs between every entity, templates are mainly defined by the following field values:

    **1. `spec.type`**: The type created by the template should match the template's
    
    **2. `spec.parameters` and `spec.steps`**: Each `parameter` is a variable that has to be filled by the user of the template when generating a new entity. A parameter can be a `Step`or a list of multiple `Steps`. 
    
    - A `Step` is a [JSONSchema](https://github.com/rjsf-team/react-jsonschema-form) with styling options, you can find further documentation on how `JSONSchema` works in their [official documentation](https://react-jsonschema-form.readthedocs.io/) however, it can also be a `uiSchema`. Both have more examples available [here](https://rjsf-team.github.io/react-jsonschema-form) and [here](https://backstage.io/docs/features/software-templates/writing-templates).

    **3. `spec.owner`**: A template should be owned by an entity `User` or `Group` responsible for maintaining Portal's overall infrastructure. Teams themselves can submit new templates to be reviewed and approved, but ultimately it should be owned by a Portal's core team.

    **4. `metadata.tags`**: It is the only relevant non-Kubernetes related `metadata` field relative to templates.


```json title="schemas.json" linenums="1"
// jsonSchema:
{
  "title": "A registration form",
  "description": "A simple form example.",
  "type": "object",
  "required": [
    "developerName",
    "githubUser",
    "email"
  ],
  "properties": {
    "developerName": {
      "type": "string",
      "title": "Developer Name",
      "default": ""
    },
    "githubUser": {
      "type": "string",
      "title": "Github User",
      "minLength": 6
    },
    "email": {
      "type": "string",
      "title": "Email",
      "minLength": 8
    }
  }
}
// uiSchema:
{
  "developerName": {
    "ui:autofocus": true,
    "ui:emptyValue": "",
    "ui:autocomplete": "first-name"
  },
  "githubUser": {
    "ui:options":{
      "orderable": false
    "ui:autofocus": true,
    "ui:emptyValue": "",
    }
  },
  "email": {
    "ui:options": {
      "inputType": "email"
    }
  }
}
```


### Other entities

<sub> (External) </sub>

As these entities have a very simple and deterministic nature when being defined, the documentation does not differ from Backstage's instructions significantly. The documentation on how to define them can be found on the references below:

- [Locations](https://backstage.io/docs/features/software-catalog/descriptor-format#kind-location)
- [Groups](https://backstage.io/docs/features/software-catalog/descriptor-format#kind-group)
- [Users](https://backstage.io/docs/features/software-catalog/descriptor-format#kind-user)
- [Systems](https://backstage.io/docs/features/software-catalog/descriptor-format#kind-system)
- [Domains](https://backstage.io/docs/features/software-catalog/descriptor-format#kind-domain)

<br>

<!-- # Plugins (Not implemented as of now (11/21/2021)

The objective of this section is to assist in configuring each plugin to each new entity being provisioned.

## Kubernetes

There are two main ways of registering entities to Kubernetes clusters. The label selector takes precedence over the entity's annotations.

*   **Labeling Kubernetes components:**

It has to apply the label `backstage.io/kubernetes-id` directly to the Kubernetes object metadata. An example can be observed below:

```yaml
'backstage.io/kubernetes-id': <BACKSTAGE_ENTITY_NAME>
```

*   **Entity annotation:**

The annotation has to be defined to on the entity's `catalog-info.yaml` definition:

```yaml
annotations:
  'backstage.io/kubernetes-id': dice-roller
``` -->
