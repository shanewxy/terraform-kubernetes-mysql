# Kubernetes MySQL Service

Terraform module which deploys containerized MySQL on Kubernetes, powered by [Bitnami Charts/MySQL](https://github.com/bitnami/charts/tree/main/bitnami/mysql).

## Usage

```hcl
module "example" {
  source = "..."

  infrastructure = {
    namespace = "default"
  }

  deployment = {
    architecture = "replication"  # i.e. standalone, replication
    version = "8.2.0"             # https://hub.docker.com/r/bitnami/mysql/tags
    username = "myuser"
    password = "mypasswd"         # requirement
    database = "mydb"
  }
}
```

## Examples

- [Replication](./examples/replication)
- [Standalone](./examples/standalone)

## Contributing

Please read our [contributing guide](./docs/CONTRIBUTING.md) if you're interested in contributing to Walrus template.

<!-- BEGIN_TF_DOCS -->
## Requirements

| Name | Version |
|------|---------|
| <a name="requirement_terraform"></a> [terraform](#requirement\_terraform) | >= 1.0 |
| <a name="requirement_helm"></a> [helm](#requirement\_helm) | >= 2.11.0 |
| <a name="requirement_kubernetes"></a> [kubernetes](#requirement\_kubernetes) | >= 2.23.0 |
| <a name="requirement_random"></a> [random](#requirement\_random) | >= 3.5.1 |

## Providers

| Name | Version |
|------|---------|
| <a name="provider_helm"></a> [helm](#provider\_helm) | >= 2.11.0 |
| <a name="provider_kubernetes"></a> [kubernetes](#provider\_kubernetes) | >= 2.23.0 |
| <a name="provider_random"></a> [random](#provider\_random) | >= 3.5.1 |

## Modules

No modules.

## Resources

| Name | Type |
|------|------|
| [helm_release.mysql](https://registry.terraform.io/providers/hashicorp/helm/latest/docs/resources/release) | resource |
| [kubernetes_config_map_v1.text_seeding](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs/resources/config_map_v1) | resource |
| [kubernetes_persistent_volume_claim_v1.url_seeding](https://registry.terraform.io/providers/hashicorp/kubernetes/latest/docs/resources/persistent_volume_claim_v1) | resource |
| [random_string.name_suffix](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/string) | resource |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_context"></a> [context](#input\_context) | Receive contextual information. When Walrus deploys, Walrus will inject specific contextual information into this field.<br><br>Examples:<pre>context:<br>  project:<br>    name: string<br>    id: string<br>  environment:<br>    name: string<br>    id: string<br>  resource:<br>    name: string<br>    id: string</pre> | `map(any)` | `{}` | no |
| <a name="input_deployment"></a> [deployment](#input\_deployment) | Specify the deployment action, including architecture and account.<br><br>Examples:<pre>deployment:<br>  version: string, optional      # https://hub.docker.com/r/bitnami/mysql/tags<br>  type: string, optional         # i.e. standalone, replication<br>  username: string, optional<br>  password: string<br>  database: string, optional</pre> | <pre>object({<br>    version  = optional(string, "8.0")<br>    type     = optional(string, "standalone")<br>    username = optional(string, "root")<br>    password = string<br>    database = optional(string, "mydb")<br>  })</pre> | n/a | yes |
| <a name="input_infrastructure"></a> [infrastructure](#input\_infrastructure) | Specify the infrastructure information for deploying.<br><br>Examples:<pre>infrastructure:<br>  namespace: string, optional<br>  image_registry: string, optional</pre> | <pre>object({<br>    namespace      = optional(string)<br>    image_registry = optional(string, "registry-1.docker.io")<br>  })</pre> | `{}` | no |
| <a name="input_replication"></a> [replication](#input\_replication) | Specify the configuration of replication deployment type.<br><br>Examples:<pre>replication:                     # two instances: one primary, one read-only secondary (same az)<br>  primary:<br>    resources:<br>      requests:<br>        cpu: number     <br>        memory: number           # in megabyte<br>      limits:<br>        cpu: number<br>        memory: number           # in megabyte<br>    storage:                     # convert to empty dir if null<br>      type: ephemeral/persistent<br>      ephemeral:                 # convert to volume claim template<br>        class: string<br>        access_mode: string<br>        size: number, optional   # in megabyte<br>      persistent:<br>        name: string             # convert to persistent volume claim<br>  secondary:<br>    resources:<br>      requests:<br>        cpu: number     <br>        memory: number           # in megabyte<br>      limits:<br>        cpu: number<br>        memory: number           # in megabyte<br>    storage:                     # convert to empty dir if null<br>      type: ephemeral/persistent<br>      ephemeral:                 # convert to volume claim template<br>        class: string<br>        access_mode: string<br>        size: number, optional   # in megabyte<br>      persistent:                # convert to persistent volume claim<br>        name: string</pre> | <pre>object({<br>    primary = optional(object({<br>      resources = optional(object({<br>        requests = object({<br>          cpu    = optional(number, 0.25)<br>          memory = optional(number, 256)<br>        })<br>        limits = optional(object({<br>          cpu    = optional(number, 0)<br>          memory = optional(number, 0)<br>        }))<br>      }), { requests = { cpu = 0.25, memory = 256 } })<br>      storage = optional(object({<br>        type = optional(string, "ephemeral")<br>        ephemeral = optional(object({<br>          class       = optional(string)<br>          access_mode = optional(string, "ReadWriteOnce")<br>          size        = optional(number)<br>        }))<br>        persistent = optional(object({<br>          name = string<br>        }))<br>      }))<br>    }), { requests = { cpu = 0.25, memory = 256 } })<br>    secondary = optional(object({<br>      resources = optional(object({<br>        requests = object({<br>          cpu    = optional(number, 0.25)<br>          memory = optional(number, 256)<br>        })<br>        limits = optional(object({<br>          cpu    = optional(number, 0)<br>          memory = optional(number, 0)<br>        }))<br>      }), { requests = { cpu = 0.25, memory = 256 } })<br>      storage = optional(object({<br>        type = optional(string, "ephemeral")<br>        ephemeral = optional(object({<br>          class       = optional(string)<br>          access_mode = optional(string, "ReadWriteOnce")<br>          size        = optional(number)<br>        }))<br>        persistent = optional(object({<br>          name = string<br>        }))<br>      }))<br>    }), { requests = { cpu = 0.25, memory = 256 } })<br>  })</pre> | `{}` | no |
| <a name="input_seeding"></a> [seeding](#input\_seeding) | Specify the configuration to seed the database after first time creating,<br>this action needs admin permission, like root user.<br><br>Examples:<pre>seeding:<br>  url:                           # need a persistent volume to store the content<br>    location: string<br>    storage:<br>      class: string, optional<br>      size: number, optional     # in megabyte<br>  text:                          # convert to configmap, only support 4kb content<br>    content: string</pre> | <pre>object({<br>    type = optional(string, "url")<br>    url = optional(object({<br>      location = string<br>      storage = optional(object({<br>        class = optional(string)<br>        size  = optional(number, 5 * 1024)<br>      }))<br>    }))<br>    text = optional(object({<br>      content = string<br>    }))<br>  })</pre> | `{}` | no |
| <a name="input_standalone"></a> [standalone](#input\_standalone) | Specify the configuration of standalone deployment type.<br><br>Examples:<pre>standalone:                      # one instance<br>  resources:<br>    requests:<br>      cpu: number     <br>      memory: number             # in megabyte<br>    limits:<br>      cpu: number<br>      memory: number             # in megabyte<br>  storage:                       # convert to empty dir if null<br>    type: ephemeral/persistent<br>    ephemeral:                   # convert to volume claim template<br>      class: string<br>      access_mode: string<br>      size: number, optional     # in megabyte<br>    persistent:                  # convert to persistent volume claim<br>      name: string</pre> | <pre>object({<br>    resources = optional(object({<br>      requests = object({<br>        cpu    = optional(number, 0.25)<br>        memory = optional(number, 256)<br>      })<br>      limits = optional(object({<br>        cpu    = optional(number, 0)<br>        memory = optional(number, 0)<br>      }))<br>    }), { requests = { cpu = 0.25, memory = 256 } })<br>    storage = optional(object({<br>      type = optional(string, "ephemeral")<br>      ephemeral = optional(object({<br>        class       = optional(string)<br>        access_mode = optional(string, "ReadWriteOnce")<br>        size        = optional(number)<br>      }))<br>      persistent = optional(object({<br>        name = string<br>      }))<br>    }))<br>  })</pre> | `{}` | no |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_context"></a> [context](#output\_context) | The input context, a map, which is used for orchestration. |
| <a name="output_database"></a> [database](#output\_database) | The name of database to access. |
| <a name="output_endpoint_internal"></a> [endpoint\_internal](#output\_endpoint\_internal) | The internal endpoints, a string list, which are used for internal access. |
| <a name="output_endpoint_internal_readonly"></a> [endpoint\_internal\_readonly](#output\_endpoint\_internal\_readonly) | The internal readonly endpoints, a string list, which are used for internal readonly access. |
| <a name="output_password"></a> [password](#output\_password) | The password of the account to access the database. |
| <a name="output_username"></a> [username](#output\_username) | The username of the account to access the database. |
<!-- END_TF_DOCS -->

## License

Copyright (c) 2023 [Seal, Inc.](https://seal.io)

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at [LICENSE](./LICENSE) file for details.

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
