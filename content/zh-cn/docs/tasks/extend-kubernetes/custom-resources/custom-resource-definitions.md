---
title: 使用 CustomResourceDefinition 扩展 Kubernetes API
content_type: task
min-kubernetes-server-version: 1.16
weight: 20
---
<!--
title: Extend the Kubernetes API with CustomResourceDefinitions
reviewers:
- deads2k
- jpbetz
- liggitt
- roycaihw
- sttts
content_type: task
min-kubernetes-server-version: 1.16
weight: 20
-->

<!-- overview -->
<!--
This page shows how to install a
[custom resource](/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
into the Kubernetes API by creating a
[CustomResourceDefinition](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#customresourcedefinition-v1-apiextensions-k8s-io).
-->
本页展示如何使用
[CustomResourceDefinition](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#customresourcedefinition-v1-apiextensions-k8s-io)
将
[定制资源（Custom Resource）](/zh-cn/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
安装到 Kubernetes API 上。

## {{% heading "prerequisites" %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

<!--
If you are using an older version of Kubernetes that is still supported, switch to
the documentation for that version to see advice that is relevant for your cluster.
-->
如果你在使用较老的、仍处于被支持范围的 Kubernetes 版本，请切换到该版本的
文档查看对于的集群而言有用的建议。

<!-- steps -->

<!--
## Create a CustomResourceDefinition

When you create a new CustomResourceDefinition (CRD), the Kubernetes API Server
creates a new RESTful resource path for each version you specify. The CRD can be
either namespaced or cluster-scoped, as specified in the CRD's `scope` field. As
with existing built-in objects, deleting a namespace deletes all custom objects
in that namespace. CustomResourceDefinitions themselves are non-namespaced and
are available to all namespaces.

For example, if you save the following CustomResourceDefinition to `resourcedefinition.yaml`:
-->
## 创建 CustomResourceDefinition  {#create-a-customresourcedefinition}

当你创建新的 CustomResourceDefinition（CRD）时，Kubernetes API 服务器会为你所
指定的每一个版本生成一个 RESTful 的 资源路径。CRD 可以是名字空间作用域的，也可以
是集群作用域的，取决于 CRD 的 `scope` 字段设置。和其他现有的内置对象一样，删除
一个名字空间时，该名字空间下的所有定制对象也会被删除。CustomResourceDefinition
本身是不受名字空间限制的，对所有名字空间可用。

例如，如果你将下面的 CustomResourceDefinition 保存到 `resourcedefinition.yaml`
文件：

<!--
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: crontabs.stable.example.com
spec:
  # group name to use for REST API: /apis/<group>/<version>
  group: stable.example.com
  # list of versions supported by this CustomResourceDefinition
  versions:
    - name: v1
      # Each version can be enabled/disabled by Served flag.
      served: true
      # One and only one version must be marked as the storage version.
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  # either Namespaced or Cluster
  scope: Namespaced
  names:
    # plural name to be used in the URL: /apis/<group>/<version>/<plural>
    plural: crontabs
    # singular name to be used as an alias on the CLI and for display
    singular: crontab
    # kind is normally the PascalCased singular type. Your resource manifests use this.
    kind: CronTab
    # shortNames allow shorter string to match your resource on the CLI
    shortNames:
    - ct
```
-->
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # 名字必需与下面的 spec 字段匹配，并且格式为 '<名称的复数形式>.<组名>'
  name: crontabs.stable.example.com
spec:
  # 组名称，用于 REST API: /apis/<组>/<版本>
  group: stable.example.com
  # 列举此 CustomResourceDefinition 所支持的版本
  versions:
    - name: v1
      # 每个版本都可以通过 served 标志来独立启用或禁止
      served: true
      # 其中一个且只有一个版本必需被标记为存储版本
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  # 可以是 Namespaced 或 Cluster
  scope: Namespaced
  names:
    # 名称的复数形式，用于 URL：/apis/<组>/<版本>/<名称的复数形式>
    plural: crontabs
    # 名称的单数形式，作为命令行使用时和显示时的别名
    singular: crontab
    # kind 通常是单数形式的帕斯卡编码（PascalCased）形式。你的资源清单会使用这一形式。
    kind: CronTab
    # shortNames 允许你在命令行使用较短的字符串来匹配资源
    shortNames:
    - ct
```

<!--
and create it:
-->
之后创建它：

```shell
kubectl apply -f resourcedefinition.yaml
```

<!--
Then a new namespaced RESTful API endpoint is created at:
-->
这样一个新的受名字空间约束的 RESTful API 端点会被创建在：

```
/apis/stable.example.com/v1/namespaces/*/crontabs/...
```

<!--
This endpoint URL can then be used to create and manage custom objects.
The `kind` of these objects will be `CronTab` from the spec of the
CustomResourceDefinition object you created above.

It might take a few seconds for the endpoint to be created.
You can watch the `Established` condition of your CustomResourceDefinition
to be true or watch the discovery information of the API server for your
resource to show up.
-->
此端点 URL 自此可以用来创建和管理定制对象。对象的 `kind` 将是来自你上面创建时
所用的 spec 中指定的 `CronTab`。

创建端点的操作可能需要几秒钟。你可以监测你的 CustomResourceDefinition 的
`Established` 状况变为 true，或者监测 API 服务器的发现信息等待你的资源出现在
那里。

<!--
## Create custom objects

After the CustomResourceDefinition object has been created, you can create
custom objects. Custom objects can contain custom fields. These fields can
contain arbitrary JSON.
In the following example, the `cronSpec` and `image` custom fields are set in a
custom object of kind `CronTab`.  The kind `CronTab` comes from the spec of the
CustomResourceDefinition object you created above.

If you save the following YAML to `my-crontab.yaml`:
-->
## 创建定制对象   {#create-custom-objects}

在创建了 CustomResourceDefinition 对象之后，你可以创建定制对象（Custom
Objects）。定制对象可以包含定制字段。这些字段可以包含任意的 JSON 数据。
在下面的例子中，在类别为 `CrontTab` 的定制对象中，设置了`cronSpec` 和 `image`
定制字段。类别 `CronTab` 来自你在上面所创建的 CRD 的规约。

如果你将下面的 YAML 保存到 `my-crontab.yaml`：

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
```

<!--
and create it:
-->
并执行创建命令：

```shell
kubectl apply -f my-crontab.yaml
```

<!--
You can then manage your CronTab objects using kubectl. For example:
-->
你就可以使用 kubectl 来管理你的 CronTab 对象了。例如：

```shell
kubectl get crontab
```

<!--
Should print a list like this:
-->
应该会输出如下列表：

```none
NAME                 AGE
my-new-cron-object   6s
```

<!--
Resource names are not case-sensitive when using kubectl, and you can use either
the singular or plural forms defined in the CRD, as well as any short names.

You can also view the raw YAML data:
-->
使用 kubectl 时，资源名称是大小写不敏感的，而且你既可以使用 CRD 中所定义的单数
形式或复数形式，也可以使用其短名称：

```shell
kubectl get ct -o yaml
```

<!--
You should see that it contains the custom `cronSpec` and `image` fields
from the YAML you used to create it:
-->
你可以看到输出中包含了你创建定制对象时在 YAML 文件中指定的定制字段 `cronSpec`
和 `image`：

```yaml
apiVersion: v1
items:
- apiVersion: stable.example.com/v1
  kind: CronTab
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"stable.example.com/v1","kind":"CronTab","metadata":{"annotations":{},"name":"my-new-cron-object","namespace":"default"},"spec":{"cronSpec":"* * * * */5","image":"my-awesome-cron-image"}}
    creationTimestamp: "2021-06-20T07:35:27Z"
    generation: 1
    name: my-new-cron-object
    namespace: default
    resourceVersion: "1326"
    uid: 9aab1d66-628e-41bb-a422-57b8b3b1f5a9
  spec:
    cronSpec: '* * * * */5'
    image: my-awesome-cron-image
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

<!--
## Delete a CustomResourceDefinition

When you delete a CustomResourceDefinition, the server will uninstall the RESTful API endpoint
and delete all custom objects stored in it.
-->
## 删除 CustomResourceDefinition    {#delete-a-customresourcedefinition}

当你删除某 CustomResourceDefinition 时，服务器会卸载其 RESTful API
端点，并删除服务器上存储的所有定制对象。

```shell
kubectl delete -f resourcedefinition.yaml
kubectl get crontabs
```

```none
Error from server (NotFound): Unable to list {"stable.example.com" "v1" "crontabs"}: the server could not find the requested resource (get crontabs.stable.example.com)
```

<!--
If you later recreate the same CustomResourceDefinition, it will start out empty.
-->
如果你在以后创建相同的 CustomResourceDefinition 时，该 CRD 会是一个空的结构。

<!--
## Specifying a structural schema

CustomResources store structured data in custom fiels (alongside the built-in
fields `apiVersion`, `kind` and `metadata`, which the API server validates
implicitly). With [OpenAPI v3.0 validation](#validation) a schema can be
specified, which is validated during creation and updates, compare below for
details and limits of such a schema.

With `apiextensions.k8s.io/v1` the definition of a structural schema is
mandatory for CustomResourceDefinitions. In the beta version of
CustomResourceDefinition, structural schemas were optional.
-->
## 设置结构化的模式   {#specifying-a-structural-schema}

CustomResource 对象在定制字段中保存结构化的数据，这些字段和内置的字段
`apiVersion`、`kind` 和 `metadata` 等一起存储，不过内置的字段都会被 API
服务器隐式完成合法性检查。有了 [OpenAPI v3.0 检查](#validation)
能力之后，你可以设置一个模式（Schema），在创建和更新定制对象时，这一模式会被用来
对对象内容进行合法性检查。参阅下文了解这类模式的细节和局限性。

在 `apiextensions.k8s.io/v1` 版本中，CustomResourceDefinition 的这一结构化模式
定义是必需的。
在 CustomResourceDefinition 的 beta 版本中，结构化模式定义是可选的。

<!--
A structural schema is an [OpenAPI v3.0 validation schema](#validation) which:

1. specifies a non-empty type (via `type` in OpenAPI) for the root, for each specified field of an object node (via `properties` or `additionalProperties` in OpenAPI) and for each item in an array node (via `items` in OpenAPI), with the exception of:
   * a node with `x-kubernetes-int-or-string: true`
   * a node with `x-kubernetes-preserve-unknown-fields: true`
2. for each field in an object and each item in an array which is specified within any of `allOf`, `anyOf`, `oneOf` or `not`, the schema also specifies the field/item outside of those logical junctors (compare example 1 and 2).
3. does not set `description`, `type`, `default`, `additionalProperties`, `nullable` within an `allOf`, `anyOf`, `oneOf` or `not`, with the exception of the two pattern for `x-kubernetes-int-or-string: true` (see below).
4. if `metadata` is specified, then only restrictions on `metadata.name` and `metadata.generateName` are allowed.
-->
结构化模式本身是一个 [OpenAPI v3.0 验证模式](#validation)，其中：

1. 为对象根（root）设置一个非空的 type 值（藉由 OpenAPI 中的 `type`），对每个
   object 节点的每个字段（藉由 OpenAPI 中的 `properties` 或 `additionalProperties`）以及
   array 节点的每个条目（藉由 OpenAPI 中的 `items`）也要设置非空的 type 值，
   除非：
   * 节点包含属性 `x-kubernetes-int-or-string: true`
   * 节点包含属性 `x-kubernetes-preserve-unknown-fields: true`
2. 对于 object 的每个字段或 array 中的每个条目，如果其定义中包含 `allOf`、`anyOf`、`oneOf`
   或 `not`，则模式也要指定这些逻辑组合之外的字段或条目（试比较例 1 和例 2)。
3. 在 `allOf`、`anyOf`、`oneOf` 或 `not` 上下文内不设置 `description`、`type`、`default`、
   `additionalProperties` 或者 `nullable`。此规则的例外是
   `x-kubernetes-int-or-string` 的两种模式（见下文）。
4. 如果 `metadata` 被设置，则只允许对 `metadata.name` 和 `metadata.generateName` 设置约束。

<!--
Non-structural example 1:
-->
非结构化的例 1:

```yaml
allOf:
- properties:
    foo:
      ...
```

<!--
conflicts with rule 2. The following would be correct:
-->
违反了第 2 条规则。下面的是正确的：

```yaml
properties:
  foo:
    ...
allOf:
- properties:
    foo:
      ...
```

<!--
Non-structural example 2:
-->
非结构化的例 2：

```yaml
allOf:
- items:
    properties:
      foo:
        ...
```

<!--
conflicts with rule 2. The following would be correct:
-->
违反了第 2 条规则。下面的是正确的：

```yaml
items:
  properties:
    foo:
      ...
allOf:
- items:
    properties:
      foo:
        ...
```

<!--
Non-structural example 3:
-->
非结构化的例 3：

```yaml
properties:
  foo:
    pattern: "abc"
  metadata:
    type: object
    properties:
      name:
        type: string
        pattern: "^a"
      finalizers:
        type: array
        items:
          type: string
          pattern: "my-finalizer"
anyOf:
- properties:
    bar:
      type: integer
      minimum: 42
  required: ["bar"]
  description: "foo bar object"
```

<!--
is not a structural schema because of the following violations:

* the type at the root is missing (rule 1).
* the type of `foo` is missing (rule 1).
* `bar` inside of `anyOf` is not specified outside (rule 2).
* `bar`'s `type` is within `anyOf` (rule 3).
* the description is set within `anyOf` (rule 3).
* `metadata.finalizers` might not be restricted (rule 4).
-->
不是一个结构化的模式，因为其中存在以下违例：

* 根节点缺失 type 设置（规则 1）
* `foo` 的 type 缺失（规则 1）
* `anyOf` 中的 `bar` 未在外部指定（规则 2）
* `bar` 的 `type` 位于 `anyOf` 中（规则 3）
* `anyOf` 中设置了 `description` （规则 3）
* `metadata.finalizers` 不可以被限制 (规则 4）

<!--
In contrast, the following, corresponding schema is structural:
-->
作为对比，下面的 YAML 所对应的模式则是结构化的：

```yaml
type: object
description: "foo bar object"
properties:
  foo:
    type: string
    pattern: "abc"
  bar:
    type: integer
  metadata:
    type: object
    properties:
      name:
        type: string
        pattern: "^a"
anyOf:
- properties:
    bar:
      minimum: 42
  required: ["bar"]
```

<!--
Violations of the structural schema rules are reported in the `NonStructural` condition in the CustomResourceDefinition.
-->
如果违反了结构化模式规则，CustomResourceDefinition 的 `NonStructural` 状况中
会包含报告信息。

<!--
### Field pruning

CustomResourceDefinitions store validated resource data in the cluster's persistence store, {{< glossary_tooltip term_id="etcd" text="etcd">}}. As with native Kubernetes resources such as {{< glossary_tooltip text="ConfigMap" term_id="configmap" >}}, if you specify a field that the API server does not recognize, the unknown field  is _pruned_ (removed) before being persisted.
-->
### 字段剪裁     {#field-pruning}

CustomResourceDefinition 在集群的持久性存储
{{< glossary_tooltip term_id="etcd" text="etcd">}}
中保存经过合法性检查的资源数据。
就像原生的 Kubernetes 资源，例如 {{< glossary_tooltip text="ConfigMap" term_id="configmap" >}}，
如果你指定了 API 服务器所无法识别的字段，则该未知字段会在保存资源之前
被 _剪裁（Pruned）_ 掉（删除）。

<!--
CRDs converted from `apiextensions.k8s.io/v1beta1` to
`apiextensions.k8s.io/v1` might lack structural schemas, and
`spec.preserveUnknownFields` might be `true`.

For legacy CustomResourceDefinition objects created as
`apiextensions.k8s.io/v1beta1` with `spec.preserveUnknownFields` set to
`true`, the following is also true:

* Pruning is not enabled.
* You can store arbitrary data.

For compatibility with `apiextensions.k8s.io/v1`, update your custom
resource definitions to:

1. Use a structural OpenAPI schema.
2. Set `spec.preserveUnknownFields` to `false`.
-->
{{< note >}}
从 `apiextensions.k8s.io/v1beta1` 转换到 `apiextensions.k8s.io/v1` 的 CRD
可能没有结构化的模式定义，因此其 `spec.preserveUnknownFields` 可能为 `true`。

对于使用 `apiextensions.k8s.io/v1beta1` 且将 `spec.preserveUnknownFields` 设置为 `true`
创建的旧 CustomResourceDefinition 对象，有以下表现：

* 裁剪未启用。
* 可以存储任意数据。

为了与 `apiextensions.k8s.io/v1` 兼容，将你的定制资源定义更新为：

1. 使用结构化的 OpenAPI 模式。
2. `spec.preserveUnknownFields` 设置为 `false`。

{{< /note >}}

<!--
If you save the following YAML to `my-crontab.yaml`:
-->
如果你将下面的 YAML 保存到 `my-crontab.yaml` 文件：

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
  someRandomField: 42
```

<!--
and create it:
-->
并创建之：

```shell
kubectl create --validate=false -f my-crontab.yaml -o yaml
```

<!--
your output is similar to:
-->
输出类似于：

```console
apiVersion: stable.example.com/v1
kind: CronTab
metadata:
  creationTimestamp: 2017-05-31T12:56:35Z
  generation: 1
  name: my-new-cron-object
  namespace: default
  resourceVersion: "285"
  uid: 9423255b-4600-11e7-af6a-28d2447dc82b
spec:
  cronSpec: '* * * * */5'
  image: my-awesome-cron-image
```

<!--
Notice that the field `someRandomField` was pruned.
-->
注意其中的字段 `someRandomField` 已经被剪裁掉。

<!--
This example turned off client-side validation to demonstrate the API server's behavior, by adding the `--validate=false` command line option.
Because the [OpenAPI validation schemas are also published](#publish-validation-schema-in-openapi-v2)
to clients, `kubectl` also checks for unknown fields and rejects those objects well before they would be sent to the API server.
-->
本例中通过 `--validate=false` 命令行选项 关闭了客户端的合法性检查以展示 API 服务器的行为，
因为 [OpenAPI 合法性检查模式也会发布到](#publish-validation-schema-in-openapi-v2)
客户端，`kubectl` 也会检查未知的字段并在对象被发送到 API
服务器之前就拒绝它们。

<!--
#### Controlling pruning

By default, all unspecified fields for a custom resource, across all versions, are pruned. It is possible though to opt-out of that for specific sub-trees of fields by adding `x-kubernetes-preserve-unknown-fields: true` in the [structural OpenAPI v3 validation schema](#specifying-a-structural-schema).
For example:
-->
#### 控制剪裁   {#controlling-pruning}

默认情况下，定制资源的所有版本中的所有未规定的字段都会被剪裁掉。
通过在结构化的 OpenAPI v3 [检查模式定义](#specifying-a-structural-schema)
中为特定字段的子树添加 `x-kubernetes-preserve-unknown-fields: true` 属性，可以
选择不对其执行剪裁操作。
例如：

```yaml
type: object
properties:
  json:
    x-kubernetes-preserve-unknown-fields: true
```

<!--
The field `json` can store any JSON value, without anything being pruned.

You can also partially specify the permitted JSON; for example:
-->
字段 `json` 可以保存任何 JSON 值，其中内容不会被剪裁掉。

你也可以部分地指定允许的 JSON 数据格式；例如：

```yaml
type: object
properties:
  json:
    x-kubernetes-preserve-unknown-fields: true
    type: object
    description: this is arbitrary JSON
```

<!--
With this, only `object` type values are allowed.

Pruning is enabled again for each specified property (or `additionalProperties`):
-->
通过这样设置，JSON 中只能设置 `object` 类型的值。

对于所指定的每个属性（或 `additionalProperties`），剪裁会再次被启用。

```yaml
type: object
properties:
  json:
    x-kubernetes-preserve-unknown-fields: true
    type: object
    properties:
      spec:
        type: object
        properties:
          foo:
            type: string
          bar:
            type: string
```

<!--
With this, the value:
-->
对于上述定义，如果提供的数值如下：

```yaml
json:
  spec:
    foo: abc
    bar: def
    something: x
  status:
    something: x
```

<!--
is pruned to:
-->
则该值会被剪裁为：

```yaml
json:
  spec:
    foo: abc
    bar: def
  status:
    something: x
```

<!--
This means that the `something` field in the specified `spec` object is pruned, but everything outside is not.
-->
这意味着所指定的 `spec` 对象中的 `something` 字段被剪裁掉，而其外部的内容都被保留。

<!--
### IntOrString

Nodes in a schema with `x-kubernetes-int-or-string: true` are excluded from rule 1, such that the following is structural:
-->
### IntOrString

模式定义中标记了 `x-kubernetes-int-or-string: true` 的节点不受前述规则 1
约束，因此下面的定义是结构化的模式：

```yaml
type: object
properties:
  foo:
    x-kubernetes-int-or-string: true
```

<!--
Also those nodes are partially excluded from rule 3 in the sense that the
following two patterns are allowed (exactly those, without variations in order
to additional fields):
-->
此外，所有这类节点也不再受规则 3 约束，也就是说，下面两种模式是被允许的
（注意，仅限于这两种模式，不支持添加新字段的任何其他变种）：

```yaml
x-kubernetes-int-or-string: true
anyOf:
- type: integer
- type: string
...
```

和

```yaml
x-kubernetes-int-or-string: true
allOf:
- anyOf:
  - type: integer
  - type: string
- ... # zero or more
...
```

<!--
With one of those specification, both an integer and a string validate.

In [Validation Schema Publishing](#publish-validation-schema-in-openapi-v2),
`x-kubernetes-int-or-string: true` is unfolded to one of the two patterns shown above.
-->
在以上两种规约中，整数值和字符串值都会被认为是合法的。

在[合法性检查模式定义的发布时](#publish-validation-schema-in-openapi-v2)，
`x-kubernetes-int-or-string: true` 会被展开为上述两种模式之一。

### RawExtension

<!--
RawExtensions (as in `runtime.RawExtension` defined in
[k8s.io/apimachinery](https://github.com/kubernetes/apimachinery/blob/03ac7a9ade429d715a1a46ceaa3724c18ebae54f/pkg/runtime/types.go#L94))
holds complete Kubernetes objects, i.e. with `apiVersion` and `kind` fields.

It is possible to specify those embedded objects (both completely without constraints or partially specified) by setting `x-kubernetes-embedded-resource: true`. For example:
-->
RawExtensions（就像在
[k8s.io/apimachinery](https://github.com/kubernetes/apimachinery/blob/03ac7a9ade429d715a1a46ceaa3724c18ebae54f/pkg/runtime/types.go#L94)
项目中 `runtime.RawExtension` 所定义的那样）
可以保存完整的 Kubernetes 对象，也就是，其中会包含 `apiVersion` 和 `kind`
字段。

通过 `x-kubernetes-embedded-resource: true` 来设定这些嵌套对象的规约（无论是
完全无限制还是部分指定都可以）是可能的。例如：

```yaml
type: object
properties:
  foo:
    x-kubernetes-embedded-resource: true
    x-kubernetes-preserve-unknown-fields: true
```

<!--
Here, the field `foo` holds a complete object, e.g.:
-->
这里，字段 `foo` 包含一个完整的对象，例如：

```yaml
foo:
  apiVersion: v1
  kind: Pod
  spec:
    ...
```

<!--
Because `x-kubernetes-preserve-unknown-fields: true` is specified alongside,
nothing is pruned. The use of `x-kubernetes-preserve-unknown-fields: true` is
optional though.

With `x-kubernetes-embedded-resource: true`, the `apiVersion`, `kind` and `metadata` are implicitly specified and validated.
-->
由于字段上设置了 `x-kubernetes-preserve-unknown-fields: true`，其中的内容不会
被剪裁。不过，在这个语境中，`x-kubernetes-preserve-unknown-fields: true` 的
使用是可选的。

设置了 `x-kubernetes-embedded-resource: true` 之后，`apiVersion`、`kind` 和
`metadata` 都是隐式设定并隐式完成合法性验证。

<!--
## Serving multiple versions of a CRD

See [Custom resource definition versioning](/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definition-versioning/)
for more information about serving multiple versions of your
CustomResourceDefinition and migrating your objects from one version to another.
-->
## 提供 CRD 的多个版本   {#serving-multiple-versions-of-a-crd}

关于如何为你的 CustomResourceDefinition 提供多个版本的支持，以及如何将你的对象
从一个版本迁移到另一个版本， 详细信息可参阅
[定制资源定义的版本](/zh-cn/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definition-versioning/)。

<!-- discussion -->

<!--
## Advanced topics

### Finalizers
-->
## 高级主题     {#advanced-topics}

### Finalizers

<!--
*Finalizers* allow controllers to implement asynchronous pre-delete hooks.
Custom objects support finalizers similar to built-in objects.

You can add a finalizer to a custom object like this:
-->
*Finalizer* 能够让控制器实现异步的删除前（Pre-delete）回调。
与内置对象类似，定制对象也支持 Finalizer。

你可以像下面一样为定制对象添加 Finalizer：

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  finalizers:
  - stable.example.com/finalizer
```

<!--
Identifiers of custom finalizers consist of a domain name, a forward slash and the name of
the finalizer. Any controller can add a finalizer to any object's list of finalizers.

The first delete request on an object with finalizers sets a value for the
`metadata.deletionTimestamp` field but does not delete it. Once this value is set,
entries in the `finalizers` list can only be removed. While any finalizers remain it is also
impossible to force the deletion of an object.

When the `metadata.deletionTimestamp` field is set, controllers watching the object execute any
finalizers they handle and remove the finalizer from the list after they are done. It is the
responsibility of each controller to remove its finalizer from the list.
-->
自定义 Finalizer 的标识符包含一个域名、一个正向斜线和 finalizer 的名称。
任何控制器都可以在任何对象的 finalizer 列表中添加新的 finalizer。

对带有 Finalizer 的对象的第一个删除请求会为其 `metadata.deletionTimestamp`
设置一个值，但不会真的删除对象。一旦此值被设置，`finalizers` 列表中的表项
只能被移除。在列表中仍然包含 finalizer 时，无法强制删除对应的对象。

当 `metadata.deletionTimestamp` 字段被设置时，监视该对象的各个控制器会
执行它们所能处理的 finalizer，并在完成处理之后将其从列表中移除。
每个控制器负责将其 finalizer 从列表中删除。

<!--
The value of `metadata.deletionGracePeriodSeconds` controls the interval between polling updates.

Once the list of finalizers is empty, meaning all finalizers have been executed, the resource is
deleted by Kubernetes.
-->
`metadata.deletionGracePeriodSeconds` 的取值控制对更新的轮询周期。

一旦 finalizers 列表为空时，就意味着所有 finalizer 都被执行过，
Kubernetes 会最终删除该资源，

<!--
### Validation

Custom resources are validated via
[OpenAPI v3 schemas](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#schemaObject),
by x-kubernetes-validations when the [Validation Rules feature](#validation-rules) is enabled, and you
can add additional validation using
[admission webhooks](/docs/reference/access-authn-authz/admission-controllers/#validatingadmissionwebhook).
-->
### 合法性检查    {#validation}

定制资源是通过
[OpenAPI v3 模式定义](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#schemaObject)
来执行合法性检查的，当启用[验证规则特性](#validation-rules)时，通过 `x-kubernetes-validations` 验证，
你可以通过使用[准入控制 Webhook](/zh-cn/docs/reference/access-authn-authz/admission-controllers/#validatingadmissionwebhook)
来添加额外的合法性检查逻辑。

<!--
Additionally, the following restrictions are applied to the schema:

- These fields cannot be set:
  - `definitions`,
  - `dependencies`,
  - `deprecated`,
  - `discriminator`,
  - `id`,
  - `patternProperties`,
  - `readOnly`,
  - `writeOnly`,
  - `xml`,
  - `$ref`.
- The field `uniqueItems` cannot be set to `true`.
- The field `additionalProperties` cannot be set to `false`.
- The field `additionalProperties` is mutually exclusive with `properties`.
-->
此外，对模式定义存在以下限制：

- 以下字段不可设置：
  - `definitions`
  - `dependencies`
  - `deprecated`
  - `discriminator`
  - `id`
  - `patternProperties`
  - `readOnly`
  - `writeOnly`
  - `xml`
  - `$ref`
- 字段 `uniqueItems` 不可设置为 `true`
- 字段 `additionalProperties` 不可设置为 `false`
- 字段 `additionalProperties` 与 `properties` 互斥，不可同时使用

<!--
The `x-kubernetes-validations` extension can be used to validate custom resources using [Common
Expression Language (CEL)](https://github.com/google/cel-spec) expressions when the [Validation
rules](#validation-rules) feature is enabled and the CustomResourceDefinition schema is a
[structural schema](#specifying-a-structural-schema).
-->
当[验证规则特性](#validation-rules)被启用并且 CustomResourceDefinition
模式是一个[结构化的模式定义](#specifying-a-structural-schema)时，
`x-kubernetes-validations` 扩展可以使用[通用表达式语言(CEL)](https://github.com/google/cel-spec)表达式来验证定制资源。

<!--
The `default` field can be set when the [Defaulting feature](#defaulting) is enabled,
which is the case with `apiextensions.k8s.io/v1` CustomResourceDefinitions.
Defaulting is in GA since 1.17 (beta since 1.16 with the `CustomResourceDefaulting`
[feature gate](/docs/reference/command-line-tools-reference/feature-gates/)
enabled, which is the case automatically for many clusters for beta features).
-->
当[设置默认值特性](#defaulting)被启用时，可以设置字段 `default`。
就 `apiextensions.k8s.io/v1` 组的 CustomResourceDefinitions，这一条件是满足的。
设置默认值的功能特性从 1.17 开始正式发布。该特性在 1.16 版本中处于
Beta 状态，要求 `CustomResourceDefaulting`
[特性门控](/zh-cn/docs/reference/command-line-tools-reference/feature-gates/)
被启用。对于大多数集群而言，Beta 状态的特性门控默认都是自动启用的。

<!--
Refer to the [structural schemas](#specifying-a-structural-schema) section for other
restrictions and CustomResourceDefinition features.
-->
关于对某些 CustomResourceDefinition 特性所必需的限制，可参见
[结构化的模式定义](#specifying-a-structural-schema)小节。

<!--
The schema is defined in the CustomResourceDefinition. In the following example, the
CustomResourceDefinition applies the following validations on the custom object:

- `spec.cronSpec` must be a string and must be of the form described by the regular expression.
- `spec.replicas` must be an integer and must have a minimum value of 1 and a maximum value of 10.

Save the CustomResourceDefinition to `resourcedefinition.yaml`:
-->
模式定义是在 CustomResourceDefinition 中设置的。在下面的例子中，
CustomResourceDefinition 对定制对象执行以下合法性检查：

- `spec.cronSpec` 必须是一个字符串，必须是正则表达式所描述的形式；
- `spec.replicas` 必须是一个整数，且其最小值为 1、最大值为 10。

将此 CustomResourceDefinition 保存到 `resourcedefinition.yaml` 文件中：

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        # openAPIV3Schema is the schema for validating custom objects.
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                  pattern: '^(\d+|\*)(/\d+)?(\s+(\d+|\*)(/\d+)?){4}$'
                image:
                  type: string
                replicas:
                  type: integer
                  minimum: 1
                  maximum: 10
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
    - ct
```

<!--
and create it:
-->
并创建 CustomResourceDefinition：

```shell
kubectl apply -f resourcedefinition.yaml
```

<!--
A request to create a custom object of kind CronTab is rejected if there are invalid values in its fields.
In the following example, the custom object contains fields with invalid values:

- `spec.cronSpec` does not match the regular expression.
- `spec.replicas` is greater than 10.

If you save the following YAML to `my-crontab.yaml`:
-->
对于一个创建 CronTab 类别对象的定制对象的请求而言，如果其字段中包含非法值，则
该请求会被拒绝。
在下面的例子中，定制对象中包含带非法值的字段：

- `spec.cronSpec` 与正则表达式不匹配
- `spec.replicas` 数值大于 10。

如果你将下面的 YAML 保存到 `my-crontab.yaml`：

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * *"
  image: my-awesome-cron-image
  replicas: 15
```

<!--
and attempt to create it:
-->
并尝试创建定制对象：

```shell
kubectl apply -f my-crontab.yaml
```

<!--
then you get an error:
-->
你会看到下面的错误信息：

```console
The CronTab "my-new-cron-object" is invalid: []: Invalid value: map[string]interface {}{"apiVersion":"stable.example.com/v1", "kind":"CronTab", "metadata":map[string]interface {}{"name":"my-new-cron-object", "namespace":"default", "deletionTimestamp":interface {}(nil), "deletionGracePeriodSeconds":(*int64)(nil), "creationTimestamp":"2017-09-05T05:20:07Z", "uid":"e14d79e7-91f9-11e7-a598-f0761cb232d1", "clusterName":""}, "spec":map[string]interface {}{"cronSpec":"* * * *", "image":"my-awesome-cron-image", "replicas":15}}:
validation failure list:
spec.cronSpec in body should match '^(\d+|\*)(/\d+)?(\s+(\d+|\*)(/\d+)?){4}$'
spec.replicas in body should be less than or equal to 10
```

<!--
If the fields contain valid values, the object creation request is accepted.

Save the following YAML to `my-crontab.yaml`:
-->
如果所有字段都包含合法值，则对象创建的请求会被接受。

将下面的 YAML 保存到 `my-crontab.yaml` 文件：

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
  replicas: 5
```

<!--
And create it:
-->
并创建定制对象：

```shell
kubectl apply -f my-crontab.yaml
crontab "my-new-cron-object" created
```
<!--
## Validation rules
-->
## 验证规则

{{< feature-state state="alpha" for_k8s_version="v1.23" >}}

<!--
Validation rules are in alpha since 1.23 and validate custom resources when the
`CustomResourceValidationExpressions` [feature
gate](/docs/reference/command-line-tools-reference/feature-gates/) is enabled.
This feature is only available if the schema is a
[structural schema](#specifying-a-structural-schema).
-->
验证规则从 1.23 开始处于 Alpha 状态，
当 `CustomResourceValidationExpressions` [特性门控](/zh-cn/docs/reference/command-line-tools-reference/feature-gates/)被启用时，
验证定制资源。这个功能只有在模式是[结构化的模式](#specifying-a-structural-schema)时才可用。

<!--
Validation rules use the [Common Expression Language (CEL)](https://github.com/google/cel-spec)
to validate custom resource values. Validation rules are included in
CustomResourceDefinition schemas using the `x-kubernetes-validations` extension.
-->
验证规则使用[通用表达式语言（CEL）](https://github.com/google/cel-spec)来验证定制资源的值。
验证规则使用 `x-kubernetes-validations` 扩展包含在 `CustomResourceDefinition` 模式定义中。

<!--
The Rule is scoped to the location of the `x-kubernetes-validations` extension in the schema.
And `self` variable in the CEL expression is bound to the scoped value.
-->
规则的作用域是模式定义中 `x-kubernetes-validations` 扩展所在的位置。
CEL 表达式中的 `self` 变量被绑定到限定作用域的取值。

<!--
All validation rules are scoped to the current object: no cross-object or stateful validation rules are supported.
-->
所有验证规则都是针对当前对象的：不支持跨对象或有状态的验证规则。

<!--
For example:
-->
例如:

```yaml
    ...
    openAPIV3Schema:
      type: object
      properties:
        spec:
          type: object
          x-kubernetes-validations:
            - rule: "self.minReplicas <= self.replicas"
              message: "replicas should be greater than or equal to minReplicas."
            - rule: "self.replicas <= self.maxReplicas"
              message: "replicas should be smaller than or equal to maxReplicas."
          properties:
            ...
            minReplicas:
              type: integer
            replicas:
              type: integer
            maxReplicas:
              type: integer
          required:
            - minReplicas
            - replicas
            - maxReplicas
```

<!--
will reject a request to create this custom resource:
-->
将拒绝创建这个定制资源的请求:

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  minReplicas: 0
  replicas: 20
  maxReplicas: 10
```
<!--
with the response:
-->

返回响应为：

```
The CronTab "my-new-cron-object" is invalid:
* spec: Invalid value: map[string]interface {}{"maxReplicas":10, "minReplicas":0, "replicas":20}: replicas should be smaller than or equal to maxReplicas.
```

<!--
`x-kubernetes-validations` could have multiple rules.

The `rule` under `x-kubernetes-validations` represents the expression which will be evaluated by CEL.

The `message` represents the message displayed when validation fails. If message is unset, the above response would be:
-->
`x-kubernetes-validations` 可以有多条规则。 

`x-kubernetes-validations` 下的 `rule` 代表将由 CEL 评估的表达式。

`message` 代表验证失败时显示的信息。如果消息没有设置，上述响应将是：
```
The CronTab "my-new-cron-object" is invalid:
* spec: Invalid value: map[string]interface {}{"maxReplicas":10, "minReplicas":0, "replicas":20}: failed rule: self.replicas <= self.maxReplicas
```

<!--
Validation rules are compiled when CRDs are created/updated.
The request of CRDs create/update will fail if compilation of validation rules fail.
Compilation process includes type checking as well.
-->
当 CRD 被创建/更新时，验证规则被编译。
如果验证规则的编译失败，CRD 的创建/更新请求将失败。
编译过程也包括类型检查。

<!--
The compilation failure:
- `no_matching_overload`: this function has no overload for the types of the arguments.

   e.g. Rule like `self == true` against a field of integer type will get error:
  ```
  Invalid value: apiextensions.ValidationRule{Rule:"self == true", Message:""}: compilation failed: ERROR: \<input>:1:6: found no matching overload for '_==_' applied to '(int, bool)'
  ```

- `no_such_field`: does not contain the desired field.

   e.g. Rule like `self.nonExistingField > 0` against a non-existing field will return the error:
  ```
  Invalid value: apiextensions.ValidationRule{Rule:"self.nonExistingField > 0", Message:""}: compilation failed: ERROR: \<input>:1:5: undefined field 'nonExistingField'
  ```

- `invalid argument`: invalid argument to macros.

  e.g. Rule like `has(self)` will return error:
  ```
  Invalid value: apiextensions.ValidationRule{Rule:"has(self)", Message:""}: compilation failed: ERROR: <input>:1:4: invalid argument to has() macro
  ```
-->
编译失败：
- `no_matching_overload`：此函数没有参数类型的重载。

  例如，像 `self == true` 这样的规则对一个整数类型的字段将得到错误：
  ```
  Invalid value: apiextensions.ValidationRule{Rule:"self == true", Message:""}: compilation failed: ERROR: \<input>:1:6: found no matching overload for '_==_' applied to '(int, bool)'
  ```

- `no_such_field`：不包含所需的字段。
  例如，针对一个不存在的字段，像 `self.nonExistingField > 0` 这样的规则将返回错误：
  ```
  Invalid value: apiextensions.ValidationRule{Rule:"self.nonExistingField > 0", Message:""}: compilation failed: ERROR: \<input>:1:5: undefined field 'nonExistingField'
  ```

- `invalid argument`：对宏的无效参数。
  例如，像 `has(self)` 这样的规则将返回错误：
  ```
  Invalid value: apiextensions.ValidationRule{Rule:"has(self)", Message:""}: compilation failed: ERROR: <input>:1:4: invalid argument to has() macro
  ```


<!--
Validation Rules Examples:

| Rule                                                                             | Purpose                                                                           |
| ----------------                                                                 | ------------                                                                      |
| `self.minReplicas <= self.replicas && self.replicas <= self.maxReplicas`         | Validate that the three fields defining replicas are ordered appropriately        |
| `'Available' in self.stateCounts`                                                | Validate that an entry with the 'Available' key exists in a map                   |
| `(size(self.list1) == 0) != (size(self.list2) == 0)`                             | Validate that one of two lists is non-empty, but not both                         |
| <code>!('MY_KEY' in self.map1) &#124;&#124; self['MY_KEY'].matches('^[a-zA-Z]*$')</code>               | Validate the value of a map for a specific key, if it is in the map               |
| `self.envars.filter(e, e.name = 'MY_ENV').all(e, e.value.matches('^[a-zA-Z]*$')` | Validate the 'value' field of a listMap entry where key field 'name' is 'MY_ENV'  |
| `has(self.expired) && self.created + self.ttl < self.expired`                    | Validate that 'expired' date is after a 'create' date plus a 'ttl' duration       |
| `self.health.startsWith('ok')`                                                   | Validate a 'health' string field has the prefix 'ok'                              |
| `self.widgets.exists(w, w.key == 'x' && w.foo < 10)`                             | Validate that the 'foo' property of a listMap item with a key 'x' is less than 10 |
| `type(self) == string ? self == '100%' : self == 1000`                           | Validate an int-or-string field for both the the int and string cases             |
| `self.metadata.name.startsWith(self.prefix)`                                     | Validate that an object's name has the prefix of another field value              |
| `self.set1.all(e, !(e in self.set2))`                                            | Validate that two listSets are disjoint                                           |
| `size(self.names) == size(self.details) && self.names.all(n, n in self.details)` | Validate the 'details' map is keyed by the items in the 'names' listSet           |

Xref: [Supported evaluation on CEL](https://github.com/google/cel-spec/blob/v0.6.0/doc/langdef.md#evaluation)
-->
验证规则例子：

| 规则                                                                             | 目的                                                                             |
| ----------------                                                                 | ------------                                                                     |
| `self.minReplicas <= self.replicas && self.replicas <= self.maxReplicas`         | 验证定义副本数的三个字段大小顺序是否正确                                             |
| `'Available' in self.stateCounts`                                                | 验证 map 中是否存在键名为 `Available`的条目                                   |
| `(size(self.list1) == 0) != (size(self.list2) == 0)`                             | 验证两个 list 之一是非空的，但不是二者都非空                                   |
| <code>!('MY_KEY' in self.map1) &#124;&#124; self['MY_KEY'].matches('^[a-zA-Z]*$')</code>               | 如果某个特定的 key 在 map 中，验证 map 中这个 key 的 value |
| `self.envars.filter(e, e.name = 'MY_ENV').all(e, e.value.matches('^[a-zA-Z]*$')` | 验证一个 listMap 中主键 'name' 为 'MY_ENV' 'value' 的表项，检查其取值 'value'             |
| `has(self.expired) && self.created + self.ttl < self.expired`    | 验证 'Expired' 日期是否晚于 'Create' 日期加上 'ttl' 持续时间                   |
| `self.health.startsWith('ok')`                                                   | 验证 'health' 字符串字段有前缀 'ok'             |
| `self.widgets.exists(w, w.key == 'x' && w.foo < 10)`                             | 验证 key 为 'x' 的 listMap 项的 'foo' 属性是否小于 10                             |
| `type(self) == string ? self == '100%' : self == 1000`                           | 在 int 型和 string 型两种情况下验证 int-or-string 字段                           |
| `self.metadata.name.startsWith(self.prefix)`                                     | 验证对象的名称是否具有另一个字段值的前缀                                         |
| `self.set1.all(e, !(e in self.set2))`                                            | 验证两个 listSet 是否不相交                                                      |
| `size(self.names) == size(self.details) && self.names.all(n, n in self.details)` | 验证 'details' map 是由 'names' listSet 的项目所决定的。                         |

参考：[CEL 中支持的求值](https://github.com/google/cel-spec/blob/v0.6.0/doc/langdef.md#evaluation)


<!--
- If the Rule is scoped to the root of a resource, it may make field selection into any fields
  declared in the OpenAPIv3 schema of the CRD as well as `apiVersion`, `kind`, `metadata.name` and
  `metadata.generateName`. This includes selection of fields in both the `spec` and `status` in the
  same expression:
-->
- 如果规则的作用域是某资源的根，则它可以对 CRD 的 OpenAPIv3 模式表达式中声明的任何字段进行字段选择，
  以及 `apiVersion`、`kind`、`metadata.name` 和 `metadata.generateName`。
  这包括在同一表达式中对 `spec` 和 `status` 的字段进行选择：
  ```yaml
      ...
      openAPIV3Schema:
        type: object
        x-kubernetes-validations:
          - rule: "self.status.availableReplicas >= self.spec.minReplicas"
        properties:
            spec:
              type: object
              properties:
                minReplicas:
                  type: integer
                ...
            status:
              type: object
              properties:
                availableReplicas:
                  type: integer
  ```

<!--
- If the Rule is scoped to an object with properties, the accessible properties of the object are field selectable
  via `self.field` and field presence can be checked via `has(self.field)`. Null valued fields are treated as
  absent fields in CEL expressions.
-->
- 如果规则的作用域是具有属性的对象，那么可以通过 `self.field` 对该对象的可访问属性进行字段选择，
  而字段存在与否可以通过 `has(self.field)` 来检查。
  在 CEL 表达式中，Null 值的字段被视为不存在的字段。

  ```yaml
      ...
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            x-kubernetes-validations:
              - rule: "has(self.foo)"
            properties:
              ...
              foo:
                type: integer
  ```

<!--
- If the Rule is scoped to an object with additionalProperties (i.e. a map) the value of the map
  are accessible via `self[mapKey]`, map containment can be checked via `mapKey in self` and all entries of the map
  are accessible via CEL macros and functions such as `self.all(...)`.
-->
- 如果规则的作用域是一个带有 additionalProperties 的对象（即map），那么 map 的值
  可以通过 `self[mapKey]` 访问，map 的包含性可以通过 `mapKey in self` 检查，
  map 中的所有条目可以通过 CEL 宏和函数如 `self.all(...)` 访问。
  ```yaml
      ...
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            x-kubernetes-validations:
              - rule: "self['xyz'].foo > 0"
            additionalProperties:
              ...
              type: object
              properties:
                foo:
                  type: integer
  ```

<!--
- If the Rule is scoped to an array, the elements of the array are accessible via `self[i]` and also by macros and
  functions.
-->
- 如果规则的作用域是 array，则 array 的元素可以通过 `self[i]` 访问，也可以通过宏和函数访问。
  ```yaml
      ...
      openAPIV3Schema:
        type: object
        properties:
          ...
          foo:
            type: array
            x-kubernetes-validations:
              - rule: "size(self) == 1"
            items:
              type: string
  ```

<!--
- If the Rule is scoped to a scalar, `self` is bound to the scalar value.
-->
- 如果规则的作用域为标量，则 `self` 将绑定到标量值。
  ```yaml
      ...
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              ...
              foo:
                type: integer
                x-kubernetes-validations:
                - rule: "self > 0"
  ```
<!--
Examples:

|type of the field rule scoped to    | Rule example             |
| -----------------------| -----------------------|
| root object            | `self.status.actual <= self.spec.maxDesired`|
| map of objects         | `self.components['Widget'].priority < 10`|
| list of integers       | `self.values.all(value, value >= 0 && value < 100)`|
| string                 | `self.startsWith('kube')`|
-->
例子：

| 规则作用域字段类型     | 规则示例             |
| -----------------------| -----------------------|
| 根对象            | `self.status.actual <= self.spec.maxDesired`|
| 对象映射         | `self.components['Widget'].priority < 10`|
| 整数列表       | `self.values.all(value, value >= 0 && value < 100)`|
| 字符串                 | `self.startsWith('kube')`|


<!--
The `apiVersion`, `kind`, `metadata.name` and `metadata.generateName` are always accessible from the root of the
object and from any x-kubernetes-embedded-resource annotated objects. No other metadata properties are accessible.
-->
`apiVersion`、`kind``metadata.name` 和 `metadata.generateName` 始终可以从对象的根目录和任何
带有 `x-kubernetes-embedded-resource` 注解的对象访问。
其他元数据属性都不可访问。

<!--
Unknown data preserved in custom resources via `x-kubernetes-preserve-unknown-fields` is not accessible in CEL
  expressions. This includes:
  - Unknown field values that are preserved by object schemas with x-kubernetes-preserve-unknown-fields.
  - Object properties where the property schema is of an "unknown type". An "unknown type" is recursively defined as:
    - A schema with no type and x-kubernetes-preserve-unknown-fields set to true
    - An array where the items schema is of an "unknown type"
    - An object where the additionalProperties schema is of an "unknown type"
-->
通过 `x-kubernetes-preserve-unknown-fields` 保存在定制资源中的未知数据在 CEL 表达中无法访问。
这包括：
  - 使用 `x-kubernetes-preserve-unknown-fields` 的对象模式保留的未知字段值。
  - 属性模式为"未知类型（Unknown Type）"的对象属性。一个"未知类型"被递归定义为：
    - 一个没有类型的模式，`x-kubernetes-preserve-unknown-fields` 设置为 true。
    - 一个数组，其中项目模式为"未知类型"
    - 一个 additionalProperties 模式为"未知类型"的对象


<!--
Only property names of the form `[a-zA-Z_.-/][a-zA-Z0-9_.-/]*` are accessible.
Accessible property names are escaped according to the following rules when accessed in the expression:
-->
只有 `[a-zA-Z_.-/][a-zA-Z0-9_.-/]*` 形式的属性名是可访问的。
当在表达式中访问时，可访问的属性名称会根据以下规则进行转义：

<!--
| ----------------------- | -----------------------|
| `__underscores__`       | `__`                  |
| `__dot__`               | `.`                   |
|`__dash__`               | `-`                   |
| `__slash__`             | `/`                   |
| `__{keyword}__`         | [CEL RESERVED keyword](https://github.com/google/cel-spec/blob/v0.6.0/doc/langdef.md#syntax)       |
-->
| 转义序列                | 属性名称等效为        |
| ----------------------- | ----------------------|
| `__underscores__`       | `__`                  |
| `__dot__`               | `.`                   |
|`__dash__`               | `-`                   |
| `__slash__`             | `/`                   |
| `__{keyword}__`         | [CEL 保留关键字](https://github.com/google/cel-spec/blob/v0.6.0/doc/langdef.md#syntax)       |

<!--
Note: CEL RESERVED keyword needs to match the exact property name to be escaped (e.g. int in the word sprint would not be escaped).
-->
注意：CEL 保留关键字需要与要转义的确切属性名匹配(例如，单词 `sprint` 中的 `int` 不会转义)。

<!--
Examples on escaping:
-->
转义的例子：

<!--
|property name    | rule with escaped property name     |
| ----------------| -----------------------             |
| namespace       | `self.__namespace__ > 0`            |
| x-prop          | `self.x__dash__prop > 0`            |
| redact__d       | `self.redact__underscores__d > 0`   |
| string          | `self.startsWith('kube')`           |
-->
|属性名           | 转义属性名规则                      |
| ----------------| -----------------------             |
| namespace       | `self.__namespace__ > 0`            |
| x-prop          | `self.x__dash__prop > 0`            |
| redact__d       | `self.redact__underscores__d > 0`   |
| string          | `self.startsWith('kube')`           |


<!--
Equality on arrays with `x-kubernetes-list-type` of `set` or `map` ignores element order, i.e. [1, 2] == [2, 1].
Concatenation on arrays with x-kubernetes-list-type use the semantics of the list type:
 - `set`: `X + Y` performs a union where the array positions of all elements in `X` are preserved and
      non-intersecting elements in `Y` are appended, retaining their partial order.
 - `map`: `X + Y` performs a merge where the array positions of all keys in `X` are preserved but the values
   are overwritten by values in `Y` when the key sets of `X` and `Y` intersect. Elements in `Y` with
   non-intersecting keys are appended, retaining their partial order.
-->
`set` 或 `map` 的 `x-Kubernetes-list-type` 的数组的等值比较会忽略元素顺序，即[1，2] == [2，1]。
使用 `x-kubernetes-list-type` 对数组进行串联时，使用 List 类型的语义：
- `set`：`X + Y` 执行一个并集操作，其中 `X` 中所有元素的数组位置被保留，
  `Y` 中不相交的元素被追加，保留其部分顺序。
- `map`：`X + Y`执行合并，其中 `X` 中所有键的数组位置被保留，
  但当 `X` 和 `Y` 的键集相交时，其值被 `Y` 中的值覆盖。
  `Y` 中键值不相交的元素被附加，保留其部分顺序。


<!--
Here is the declarations type mapping between OpenAPIv3 and CEL type:
-->
以下是 OpenAPIV3 和 CEL 类型之间的声明类型映射：

<!--
| OpenAPIv3 type                                     | CEL type                                                                                                                     |
| -------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| 'object' with Properties                           | object / "message type"                                                                                                      |
| 'object' with AdditionalProperties                 | map                                                                                                                          |
| 'object' with x-kubernetes-embedded-type           | object / "message type", 'apiVersion', 'kind', 'metadata.name' and 'metadata.generateName' are implicitly included in schema |
| 'object' with x-kubernetes-preserve-unknown-fields | object / "message type", unknown fields are NOT accessible in CEL expression                                                 |
| x-kubernetes-int-or-string                         | dynamic object that is either an int or a string, `type(value)` can be used to check the type                                |
| 'array                                             | list                                                                                                                         |
| 'array' with x-kubernetes-list-type=map            | list with map based Equality & unique key guarantees                                                                         |
| 'array' with x-kubernetes-list-type=set            | list with set based Equality & unique entry guarantees                                                                       |
| 'boolean'                                          | boolean                                                                                                                      |
| 'number' (all formats)                             | double                                                                                                                       |
| 'integer' (all formats)                            | int (64)                                                                                                                     |
| 'null'                                             | null_type                                                                                                                    |
| 'string'                                           | string                                                                                                                       |
| 'string' with format=byte (base64 encoded)         | bytes                                                                                                                        |
| 'string' with format=date                          | timestamp (google.protobuf.Timestamp)                                                                                        |
| 'string' with format=datetime                      | timestamp (google.protobuf.Timestamp)                                                                                        |
| 'string' with format=duration                      | duration (google.protobuf.Duration)                                                                                          |
-->
| OpenAPIv3 类型                                     | CEL 类型                                                                                                                     |
| -------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| 带有 Properties 的对象                           | 对象 / "消息类型"                                                                                                      |
| 带有 AdditionalProperties 的对象                 | map                                                                                                                          |
| 带有 x-kubernetes-embedded-type 的对象           | 对象 / "消息类型"，'apiVersion'、'kind'、'metadata.name' 和 'metadata.generateName' 都隐式包含在模式中 |
| 带有 x-kubernetes-preserve-unknown-fields 的对象 | 对象 / "消息类型"，未知字段无法从 CEL 表达式中访问                                                 |
| x-kubernetes-int-or-string                         | 可能是整数或字符串的动态对象，可以用 `type(value)` 来检查类型                                |
| 数组                                            | list                                                                                                                         |
| 带有 x-kubernetes-list-type=map 的数组           | 列表，基于集合等值和唯一键名保证的 map 组成                                                                          |
| 带有 x-kubernetes-list-type=set 的数组            | 列表，基于集合等值和唯一键名保证的 set 组成                                                                        |
| 布尔值                                          | boolean                                                                                                                      |
| 数字 (各种格式)                             | double                                                                                                                       |
| 整数 (各种格式)                            | int (64)                                                                                                                     |
| 'null'                                             | null_type                                                                                                                    |
| 字符串                                           | string                                                                                                                       |
| 带有 format=byte （base64 编码）字符串         | bytes                                                                                                                        |
| 带有 format=date 字符串                          | timestamp (google.protobuf.Timestamp)                                                                                        |
| 带有 format=datetime 字符串                      | timestamp (google.protobuf.Timestamp)                                                                                        |
| 带有 format=duration 字符串                      | duration (google.protobuf.Duration)                                                                                          |

<!--
xref: [CEL types](https://github.com/google/cel-spec/blob/v0.6.0/doc/langdef.md#values), [OpenAPI
types](https://swagger.io/specification/#data-types), [Kubernetes Structural Schemas](https://kubernetes.io/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#specifying-a-structural-schema).
-->
参考：[CEL 类型](https://github.com/google/cel-spec/blob/v0.6.0/doc/langdef.md#values)，
[OpenAPI 类型](https://swagger.io/specification/#data-types)，
[Kubernetes 结构化模式](/zh-cn/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#specifying-a-structural-schema)。

<!--
#### Validation functions {#available-validation-functions}
-->
#### 验证函数   {#available-validation-functions}

<!--
Functions available include:
  - CEL standard functions, defined in the [list of standard definitions](https://github.com/google/cel-spec/blob/v0.7.0/doc/langdef.md#list-of-standard-definitions)
  - CEL standard [macros](https://github.com/google/cel-spec/blob/v0.7.0/doc/langdef.md#macros)
  - CEL [extended string function library](https://pkg.go.dev/github.com/google/cel-go@v0.11.2/ext#Strings)
  - Kubernetes [CEL extension library](https://pkg.go.dev/k8s.io/apiextensions-apiserver@v0.24.0/pkg/apiserver/schema/cel/library#pkg-functions)
-->
可用的函数包括：
  - CEL 标准函数，在[标准定义列表](https://github.com/google/cel-spec/blob/v0.7.0/doc/langdef.md#list-of-standard-definitions)中定义
  - CEL 标准[宏](https://github.com/google/cel-spec/blob/v0.7.0/doc/langdef.md#macros)
  - CEL [扩展字符串函数库](https://pkg.go.dev/github.com/google/cel-go@v0.11.2/ext#Strings)
  - Kubernetes [CEL 扩展库](https://pkg.go.dev/k8s.io/apiextensions-apiserver@v0.24.0/pkg/apiserver/schema/cel/library#pkg-functions)

<!--
#### Transition rules
-->
#### 转换规则

<!--
A rule that contains an expression referencing the identifier `oldSelf` is implicitly considered a
_transition rule_. Transition rules allow schema authors to prevent certain transitions between two
otherwise valid states. For example:
-->
包含引用标识符 `oldSself` 的表达式的规则被隐式视为“转换规则（Transition Rule）”。
转换规则允许模式作者阻止两个原本有效的状态之间的某些转换。例如：

```yaml
type: string
enum: ["low", "medium", "high"]
x-kubernetes-validations:
- rule: "!(self == 'high' && oldSelf == 'low') && !(self == 'low' && oldSelf == 'high')"
  message: cannot transition directly between 'low' and 'high'
```

<!--
Unlike other rules, transition rules apply only to operations meeting the following criteria:
-->
与其他规则不同，转换规则仅适用于满足以下条件的操作：

<!--
- The operation updates an existing object. Transition rules never apply to create operations.
-->
- 更新现有对象的操作。转换规则从不适用于创建操作。

<!--
- Both an old and a new value exist. It remains possible to check if a value has been added or
  removed by placing a transition rule on the parent node. Transition rules are never applied to
  custom resource creation. When placed on an optional field, a transition rule will not apply to
  update operations that set or unset the field.
-->
- 旧的值和新的值都存在。仍然可以通过在父节点上放置转换规则来检查值是否已被添加或移除。
  转换规则从不应用于定制资源创建。当被放置在可选字段上时，转换规则将不适用于设置或取消设置该字段的更新操作。

<!--
- The path to the schema node being validated by a transition rule must resolve to a node that is
  comparable between the old object and the new object. For example, list items and their
  descendants (`spec.foo[10].bar`) can't necessarily be correlated between an existing object and a
  later update to the same object.
-->
- 被转换规则验证的模式节点的路径必须解析到一个在旧对象和新对象之间具有可比性的节点。
  例如，列表项和它们的后代（`spec.foo[10].bar`）不一定能在现有对象和后来对同一对象的更新之间产生关联。

<!--
Errors will be generated on CRD writes if a schema node contains a transition rule that can never be
applied, e.g. "*path*: update rule *rule* cannot be set on schema because the schema or its parent
schema is not mergeable".
-->
如果一个模式节点包含一个永远不能应用的转换规则，在 CRD 写入时将会产生错误，例如：
"*path*: update rule *rule* cannot be set on schema because the schema or its parent
schema is not mergeable"。

<!--
Transition rules are only allowed on _correlatable portions_ of a schema.
A portion of the schema is correlatable if all `array` parent schemas are of type `x-kubernetes-list-type=map`; any `set`or `atomic`array parent schemas make it impossible to unambiguously correlate a `self` with `oldSelf`.
-->
转换规则只允许在模式的“可关联部分（Correlatable Portions）”中使用。
如果所有 `array` 父模式都是 `x-kubernetes-list-type=map`类型的，那么该模式的一部分就是可关联的；
任何 `set` 或者 `atomic` 数组父模式都不支持确定性地将 `self` 与 `oldSelf` 关联起来。

<!--
Here are some examples for transition rules:
-->
这是一些转换规则的例子：

<!--
{{< table caption="Transition rules examples" >}}
| Use Case                                                          | Rule
| --------                                                          | --------
| Immutability                                                      | `self.foo == oldSelf.foo`
| Prevent modification/removal once assigned                        | `oldSelf != 'bar' \|\| self == 'bar'` or `!has(oldSelf.field) \|\| has(self.field)`
| Append-only set                                                   | `self.all(element, element in oldSelf)`
| If previous value was X, new value can only be A or B, not Y or Z | `oldSelf != 'X' \|\| self in ['A', 'B']`
| Monotonic (non-decreasing) counters                               | `self >= oldSelf`
{{< /table >}}
-->
{{< table caption="转换规则样例" >}}
| 用例                                                 | 规则
| --------                                             | --------
| 不可变                                               | `self.foo == oldSelf.foo`
| 赋值后禁止修改/删除                                  | `oldSelf != 'bar' \|\| self == 'bar'` or `!has(oldSelf.field) \|\| has(self.field)`
| 仅附加的 set                                         | `self.all(element, element in oldSelf)`
| 如果之前的值为 X，则新值只能为 A 或 B，不能为 Y 或 Z | `oldSelf != 'X' \|\| self in ['A', 'B']`
| 单调（非递减）计数器                                   | `self >= oldSelf`
{{< /table >}}

<!--
#### Resource use by validation functions
-->
#### 验证函数的资源使用

<!--
When you create or update a CustomResourceDefinition that uses validation rules,
the API server checks the likely impact of running those validation rules. If a rule is
estimated to be prohibitively expensive to execute, the API server rejects the create
or update operation, and returns an error message.
-->
当你创建或更新一个使用验证规则的 CustomResourceDefinition 时，
API 服务器会检查运行这些验证规则可能产生的影响。
如果一个规则的执行成本过高，API 服务器会拒绝创建或更新操作，并返回一个错误信息。
<!--
A similar system is used at runtime that observes the actions the interpreter takes. If the interpreter executes
too many instructions, execution of the rule will be halted, and an error will result.
-->
运行时也使用类似的系统来观察解释器的行动。如果解释器执行了太多的指令，规则的执行将被停止，并且会产生一个错误。
<!--
Each CustomResourceDefinition is also allowed a certain amount of resources to finish executing all of
its validation rules. If the sum total of its rules are estimated at creation time to go over that limit,
then a validation error will also occur.
-->
每个 CustomResourceDefinition 也被允许有一定数量的资源来完成其所有验证规则的执行。
如果在创建时估计其规则的总和超过了这个限制，那么也会发生验证错误。

<!--
You are unlikely to encounter issues with the resource budget for validation if you only
specify rules that always take the same amount of time regardless of how large their input is.
-->
如果你只指定那些无论输入量有多大都要花费相同时间的规则，你不太可能遇到验证的资源预算问题。
<!--
For example, a rule that asserts that `self.foo == 1` does not by itself have any
risk of rejection on validation resource budget groups.
-->
例如，一个断言 `self.foo == 1` 的规则本身不存在因为资源预算组验证而导致被拒绝的风险。
<!--
But if `foo` is a string and you define a validation rule `self.foo.contains("someString")`, that rule takes
longer to execute depending on how long `foo` is.
-->
但是，如果 `foo` 是一个字符串，而你定义了一个验证规则 `self.foo.contains("someString")`，
这个规则需要更长的时间来执行，取决于 `foo` 有多长。
<!--
Another example would be if `foo` were an array, and you specified a validation rule `self.foo.all(x, x > 5)`. The cost system always assumes the worst-case scenario if
a limit on the length of `foo` is not given, and this will happen for anything that can be iterated
over (lists, maps, etc.).
-->
另一个例子是如果 `foo` 是一个数组，而你指定了验证规则 `self.foo.all(x, x > 5)`。
如果没有给出 `foo` 的长度限制，成本系统总是假设最坏的情况，这将发生在任何可以被迭代的事物上（list、map 等）。

<!--
Because of this, it is considered best practice to put a limit via `maxItems`, `maxProperties`, and
`maxLength` for anything that will be processed in a validation rule in order to prevent validation errors during cost estimation. For example, given this schema with one rule:
-->
因此，通过 `maxItems`，`maxProperties` 和 `maxLength` 进行限制被认为是最佳实践，
以在验证规则中处理任何内容，以防止在成本估算期间验证错误。例如，给定具有一个规则的模式：

```yaml
openAPIV3Schema:
  type: object
  properties:
    foo:
      type: array
      items:
        type: string
      x-kubernetes-validations:
        - rule: "self.all(x, x.contains('a string'))"
```

<!--
then the API server rejects this rule on validation budget grounds with error:
-->
API 服务器以验证预算为由拒绝该规则，并显示错误：
```
 spec.validation.openAPIV3Schema.properties[spec].properties[foo].x-kubernetes-validations[0].rule: Forbidden:
 CEL rule exceeded budget by more than 100x (try simplifying the rule, or adding maxItems, maxProperties, and
 maxLength where arrays, maps, and strings are used)
```

<!--
The rejection happens because `self.all` implies calling `contains()` on every string in `foo`,
which in turn will check the given string to see if it contains `'a string'`. Without limits, this is a very
expensive rule.
-->
这个拒绝会发生是因为 `self.all` 意味着对 `foo` 中的每一个字符串调用 `contains()`，
而这又会检查给定的字符串是否包含 `'a string'`。如果没有限制，这是一个非常昂贵的规则。

<!--
If you do not specify any validation limit, the estimated cost of this rule will exceed the per-rule cost limit. But if you
add limits in the appropriate places, the rule will be allowed:
-->
如果你不指定任何验证限制，这个规则的估计成本将超过每条规则的成本限制。
但如果你在适当的地方添加限制，该规则将被允许：

```yaml
openAPIV3Schema:
  type: object
  properties:
    foo:
      type: array
      maxItems: 25
      items:
        type: string
        maxLength: 10
      x-kubernetes-validations:
        - rule: "self.all(x, x.contains('a string'))"
```

<!--
The cost estimation system takes into account how many times the rule will be executed in addition to the
estimated cost of the rule itself. For instance, the following rule will have the same estimated cost as the
previous example (despite the rule now being defined on the individual array items):
-->
成本评估系统除了考虑规则本身的估计成本外，还考虑到规则将被执行的次数。
例如，下面这个规则的估计成本与前面的例子相同（尽管该规则现在被定义在单个数组项上）：

```yaml
openAPIV3Schema:
  type: object
  properties:
    foo:
      type: array
      maxItems: 25
      items:
        type: string
        x-kubernetes-validations:
          - rule: "self.contains('a string'))"
        maxLength: 10
```

<!--
If a list inside of a list has a validation rule that uses `self.all`, that is significantly more expensive
than a non-nested list with the same rule. A rule that would have been allowed on a non-nested list might need lower limits set on both nested lists in order to be allowed. For example, even without having limits set,
the following rule is allowed:
-->
如果在一个列表内部的一个列表有一个使用 `self.all` 的验证规则，那就会比具有相同规则的非嵌套列表的成本高得多。
一个在非嵌套列表中被允许的规则可能需要在两个嵌套列表中设置较低的限制才能被允许。
例如，即使没有设置限制，下面的规则也是允许的：

```yaml
openAPIV3Schema:
  type: object
  properties:
    foo:
      type: array
      items:
        type: integer
    x-kubernetes-validations:
      - rule: "self.all(x, x == 5)"
```

<!--
But the same rule on the following schema (with a nested array added) produces a validation error:
-->
但是同样的规则在下面的模式中（添加了一个嵌套数组）产生了一个验证错误：

```yaml
openAPIV3Schema:
  type: object
  properties:
    foo:
      type: array
      items:
        type: array
        items:
          type: integer
        x-kubernetes-validations:
          - rule: "self.all(x, x == 5)"
```

<!--
This is because each item of `foo` is itself an array, and each subarray in turn calls `self.all`. Avoid nested
lists and maps if possible where validation rules are used.
-->
这是因为 `foo` 的每一项本身就是一个数组，而每一个子数组依次调用 `self.all`。
在使用验证规则的地方，尽可能避免嵌套的列表和字典。

<!--
### Defaulting
-->
### 设置默认值   {#efaulting}

<!--
To use defaulting, your CustomResourceDefinition must use API version `apiextensions.k8s.io/v1`.
-->
{{< note >}}
要使用设置默认值功能，你的 CustomResourceDefinition 必须使用 API 版本 `apiextensions.k8s.io/v1`。
{{< /note >}}

<!--
Defaulting allows to specify default values in the [OpenAPI v3 validation schema](#validation):
-->
设置默认值的功能允许在 [OpenAPI v3 合法性检查模式定义](#validation)中设置默认值：

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        # openAPIV3Schema 是用来检查定制对象的模式定义
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                  pattern: '^(\d+|\*)(/\d+)?(\s+(\d+|\*)(/\d+)?){4}$'
                  default: "5 0 * * *"
                image:
                  type: string
                replicas:
                  type: integer
                  minimum: 1
                  maximum: 10
                  default: 1
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
    - ct
```

<!--
With this both `cronSpec` and `replicas` are defaulted:
-->
使用此 CRD 定义时，`cronSpec` 和 `replicas` 都会被设置默认值：

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  image: my-awesome-cron-image
```

<!--
leads to
-->
会生成：

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "5 0 * * *"
  image: my-awesome-cron-image
  replicas: 1
```

<!--
Defaulting happens on the object

* in the request to the API server using the request version defaults,
* when reading from etcd using the storage version defaults,
* after mutating admission plugins with non-empty patches using the admission webhook object version defaults.

Defaults applied when reading data from etcd are not automatically written back to etcd. An update request via the API is required to persist those defaults back into etcd.
-->
默认值设定的行为发生在定制对象上：

* 在向 API 服务器发送的请求中，基于请求版本的设定设置默认值；
* 在从 etcd 读取对象时，使用存储版本来设置默认值；
* 在 Mutating 准入控制插件执行非空的补丁操作时，基于准入 Webhook 对象
  版本设置默认值。

从 etcd 中读取数据时所应用的默认值设置不会被写回到 etcd 中。
需要通过 API 执行更新请求才能将这种方式设置的默认值写回到 etcd。

<!--
Default values must be pruned (with the exception of defaults for `metadata` fields) and must validate against a provided schema.

Default values for `metadata` fields of `x-kubernetes-embedded-resources: true` nodes (or parts of a default value covering `metadata`) are not pruned during CustomResourceDefinition creation, but through the pruning step during handling of requests.
-->
默认值一定会被剪裁（除了 `metadata` 字段的默认值设置），且必须通过所提供
的模式定义的检查。

针对 `x-kubernetes-embedded-resource: true` 节点（或者包含 `metadata` 字段的结构的默认值）
的 `metadata` 字段的默认值设置不会在 CustomResourceDefinition 创建时被剪裁，
而是在处理请求的字段剪裁阶段被删除。

<!--
#### Defaulting and Nullable

**New in 1.20:** null values for fields that either don't specify the nullable flag, or give it a `false` value, will be pruned before defaulting happens. If a default is present, it will be applied. When nullable is `true`, null values will be conserved and won't be defaulted.

For example, given the OpenAPI schema below:
-->
#### 设置默认值和字段是否可为空（Nullable）   {#defaulting-and-nullable}

**1.20 版本新增:** 对于未设置其 nullable 标志的字段或者将该标志设置为
`false` 的字段，其空值（Null）会在设置默认值之前被剪裁掉。如果对应字段
存在默认值，则默认值会被赋予该字段。当 `nullable` 被设置为 `true` 时，
字段的空值会被保留，且不会在设置默认值时被覆盖。

例如，给定下面的 OpenAPI 模式定义：

```yaml
type: object
properties:
  spec:
    type: object
    properties:
      foo:
        type: string
        nullable: false
        default: "default"
      bar:
        type: string
        nullable: true
      baz:
        type: string
```

<!--
creating an object with null values for `foo` and `bar` and `baz`
-->
像下面这样创建一个为 `foo`、`bar` 和 `baz` 设置空值的对象时：

```yaml
spec:
  foo: null
  bar: null
  baz: null
```

<!--
leads to
-->
其结果会是这样：

```yaml
spec:
  foo: "default"
  bar: null
```

<!--
with `foo` pruned and defaulted because the field is non-nullable, `bar` maintaining the null value due to `nullable: true`, and `baz` pruned because the field is non-nullable and has no default.
-->
其中的 `foo` 字段被剪裁掉并重新设置默认值，因为该字段是不可为空的。
`bar` 字段的 `nullable: true` 使得其能够保有其空值。
`baz` 字段则被完全剪裁掉，因为该字段是不可为空的，并且没有默认值设置。

<!--
### Publish Validation Schema in OpenAPI v2

CustomResourceDefinition [OpenAPI v3 validation schemas](#validation) which are [structural](#specifying-a-structural-schema) and [enable pruning](#field-pruning) are published as part of the [OpenAPI v2 spec](/docs/concepts/overview/kubernetes-api/#openapi-and-swagger-definitions) from Kubernetes API server.

The [kubectl](/docs/reference/kubectl/) command-line tool consumes the published schema to perform client-side validation (`kubectl create` and `kubectl apply`), schema explanation (`kubectl explain`) on custom resources. The published schema can be consumed for other purposes as well, like client generation or documentation.
-->
### 以 OpenAPI v2 形式发布合法性检查模式      {#publish-validation-schema-in-openapi-v2}

CustomResourceDefinition 的[结构化的](#specifying-a-structural-schema)、
[启用了剪裁的](#preserving-unknown-fields) [OpenAPI v3 合法性检查模式](#validation)
会在 Kubernetes API 服务器上作为
[OpenAPI v2 规约](/zh-cn/docs/concepts/overview/kubernetes-api/#openapi-and-swagger-definitions)
的一部分发布出来。

[kubectl](/zh-cn/docs/reference/kubectl/) 命令行工具会基于所发布的模式定义来执行客户端的合法性检查（`kubectl create` 和 `kubectl apply`），为定制资源的模式定义提供解释（`kubectl explain`）。
所发布的模式还可被用于其他目的，例如生成客户端或者生成文档。

<!--
The OpenAPI v3 validation schema is converted to OpenAPI v2 schema, and
show up in `definitions` and `paths` fields in the [OpenAPI v2 spec](/docs/concepts/overview/kubernetes-api/#openapi-and-swagger-definitions).

The following modifications are applied during the conversion to keep backwards compatibility with
kubectl in previous 1.13 version. These modifications prevent kubectl from being over-strict and rejecting
valid OpenAPI schemas that it doesn't understand. The conversion won't modify the validation schema defined in CRD,
and therefore won't affect [validation](#validation) in the API server.
-->
OpenAPI v3 合法性检查模式定义会被转换为 OpenAPI v2 模式定义，并出现在
[OpenAPI v2 规范](/zh-cn/docs/concepts/overview/kubernetes-api/#openapi-and-swagger-definitions)
的 `definitions` 和 `paths` 字段中。

在转换过程中会发生以下修改，目的是保持与 1.13 版本以前的 kubectl 工具兼容。
这些修改可以避免 kubectl 过于严格，以至于拒绝它无法理解的 OpenAPI 模式定义。
转换过程不会更改 CRD 中定义的合法性检查模式定义，因此不会影响到 API 服务器中
的[合法性检查](#validation)。

<!--
1. The following fields are removed as they aren't supported by OpenAPI v2 (in future versions OpenAPI v3 will be used without these restrictions)
   - The fields `allOf`, `anyOf`, `oneOf` and `not` are removed
2. If `nullable: true` is set, we drop `type`, `nullable`, `items` and `properties` because OpenAPI v2 is not able to express nullable. To avoid kubectl to reject good objects, this is necessary.
-->
1. 以下字段会被移除，因为它们在 OpenAPI v2 中不支持（在将来版本中将使用 OpenAPI v3，
   因而不会有这些限制）
   - 字段 `allOf`、`anyOf`、`oneOf` 和 `not` 会被删除
2. 如果设置了 `nullable: true`，我们会丢弃 `type`、`nullable`、`items` 和 `properties`
   OpenAPI v2 无法表达 Nullable。为了避免 kubectl 拒绝正常的对象，这一转换是必要的。

<!--
### Additional printer columns

The kubectl tool relies on server-side output formatting. Your cluster's API server decides which
columns are shown by the `kubectl get` command. You can customize these columns for a
CustomResourceDefinition. The following example adds the `Spec`, `Replicas`, and `Age`
columns.

Save the CustomResourceDefinition to `resourcedefinition.yaml`:
-->
### 额外的打印列    {#additional-printer-columns}

`kubectl` 工具依赖服务器端的输出格式化。你的集群的 API 服务器决定 `kubectl
get` 命令要显示的列有哪些。
你可以为 CustomResourceDefinition 定制这些要打印的列。
下面的例子添加了 `Spec`、`Replicas` 和 `Age` 列：

将此 CustomResourceDefinition 保存到 `resourcedefinition.yaml` 文件：

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
    - ct
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              cronSpec:
                type: string
              image:
                type: string
              replicas:
                type: integer
    additionalPrinterColumns:
    - name: Spec
      type: string
      description: The cron spec defining the interval a CronJob is run
      jsonPath: .spec.cronSpec
    - name: Replicas
      type: integer
      description: The number of jobs launched by the CronJob
      jsonPath: .spec.replicas
    - name: Age
      type: date
      jsonPath: .metadata.creationTimestamp
```

<!--
Create the CustomResourceDefinition:
-->
创建 CustomResourceDefinition：

```shell
kubectl apply -f resourcedefinition.yaml
```

<!--
Create an instance using the `my-crontab.yaml` from the previous section.
-->
使用前文中的 `my-crontab.yaml` 创建一个实例。

<!--
Invoke the server-side printing:
-->
启用服务器端打印输出：

```shell
kubectl get crontab my-new-cron-object
```

<!--
Notice the `NAME`, `SPEC`, `REPLICAS`, and `AGE` columns in the output:
-->
注意输出中的 `NAME`、`SPEC`、`REPLICAS` 和 `AGE` 列：

```
NAME                 SPEC        REPLICAS   AGE
my-new-cron-object   * * * * *   1          7s
```

<!--
The `NAME` column is implicit and does not need to be defined in the CustomResourceDefinition.
-->
{{< note >}}
`NAME` 列是隐含的，不需要在 CustomResourceDefinition 中定义。
{{< /note >}}

<!--
#### Priority

Each column includes a `priority` field. Currently, the priority
differentiates between columns shown in standard view or wide view (using the `-o wide` flag).

- Columns with priority `0` are shown in standard view.
- Columns with priority greater than `0` are shown only in wide view.
-->
#### 优先级    {#priority}

每个列都包含一个 `priority`（优先级）字段。当前，优先级用来区分标准视图（Standard
View）和宽视图（Wide View）（使用 `-o wide` 标志）中显示的列：

- 优先级为 `0` 的列会在标准视图中显示。
- 优先级大于 `0` 的列只会在宽视图中显示。

<!--
#### Type

A column's `type` field can be any of the following (compare [OpenAPI v3 data types](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#dataTypes)):

- `integer` – non-floating-point numbers
- `number` – floating point numbers
- `string` – strings
- `boolean` – `true` or `false`
- `date` – rendered differentially as time since this timestamp.
-->
#### 类型    {#type}

列的 `type` 字段可以是以下值之一
（比较 [OpenAPI v3 数据类型](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#dataTypes)）：

- `integer` – 非浮点数字
- `number` – 浮点数字
- `string` – 字符串
- `boolean` – `true` 或 `false`
- `date` – 显示为以自此时间戳以来经过的时长

<!--
If the value inside a CustomResource does not match the type specified for the column,
the value is omitted. Use CustomResource validation to ensure that the value
types are correct.
-->
如果定制资源中的值与列中指定的类型不匹配，该值会被忽略。
你可以通过定制资源的合法性检查来确保取值类型是正确的。

<!--
#### Format

A column's `format` field can be any of the following:
-->
#### 格式    {#format}

列的 `format` 字段可以是以下值之一：

- `int32`
- `int64`
- `float`
- `double`
- `byte`
- `date`
- `date-time`
- `password`

<!--
The column's `format` controls the style used when `kubectl` prints the value.
-->
列的 `format` 字段控制 `kubectl` 打印对应取值时采用的风格。

<!--
### Subresources

Custom resources support `/status` and `/scale` subresources.

The status and scale subresources can be optionally enabled by
defining them in the CustomResourceDefinition.
-->
### 子资源     {#subresources}

定制资源支持 `/status` 和 `/scale` 子资源。

通过在 CustomResourceDefinition 中定义 `status` 和 `scale`，
可以有选择地启用这些子资源。

<!--
#### Status subresource

When the status subresource is enabled, the `/status` subresource for the custom resource is exposed.

- The status and the spec stanzas are represented by the `.status` and `.spec` JSONPaths respectively inside of a custom resource.
- `PUT` requests to the `/status` subresource take a custom resource object and ignore changes to anything except the status stanza.
- `PUT` requests to the `/status` subresource only validate the status stanza of the custom resource.
- `PUT`/`POST`/`PATCH` requests to the custom resource ignore changes to the status stanza.
- The `.metadata.generation` value is incremented for all changes, except for changes to `.metadata` or `.status`.
-->
#### Status 子资源  {#status-subresource}

当启用了 status 子资源时，对应定制资源的 `/status` 子资源会被暴露出来。

- status 和 spec 内容分别用定制资源内的 `.status` 和 `.spec` JSON 路径来表达；
- 对 `/status` 子资源的 `PUT` 请求要求使用定制资源对象作为其输入，但会忽略
  status 之外的所有内容。
- 对 `/status` 子资源的 `PUT` 请求仅对定制资源的 status 内容进行合法性检查。
- 对定制资源的 `PUT`、`POST`、`PATCH` 请求会忽略 status 内容的改变。
- 对所有变更请求，除非改变是针对 `.metadata` 或 `.status`，`.metadata.generation`
  的取值都会增加。
<!--
- Only the following constructs are allowed at the root of the CRD OpenAPI validation schema:
-->
- 在 CRD OpenAPI 合法性检查模式定义的根节点，只允许存在以下结构：

  - `description`
  - `example`
  - `exclusiveMaximum`
  - `exclusiveMinimum`
  - `externalDocs`
  - `format`
  - `items`
  - `maximum`
  - `maxItems`
  - `maxLength`
  - `minimum`
  - `minItems`
  - `minLength`
  - `multipleOf`
  - `pattern`
  - `properties`
  - `required`
  - `title`
  - `type`
  - `uniqueItems`

<!--
#### Scale subresource

When the scale subresource is enabled, the `/scale` subresource for the custom resource is exposed.
The `autoscaling/v1.Scale` object is sent as the payload for `/scale`.

To enable the scale subresource, the following fields are defined in the CustomResourceDefinition.
-->
#### Scale 子资源   {#scale-subresource}

当启用了 scale 子资源时，定制资源的 `/scale` 子资源就被暴露出来。
针对 `/scale` 所发送的对象是 `autoscaling/v1.Scale`。

为了启用 scale 子资源，CustomResourceDefinition 定义了以下字段：

<!--
- `specReplicasPath` defines the JSONPath inside of a custom resource that corresponds to `scale.spec.replicas`.

  - It is a required value.
  - Only JSONPaths under `.spec` and with the dot notation are allowed.
  - If there is no value under the `specReplicasPath` in the custom resource,
the `/scale` subresource will return an error on GET.
-->
- `specReplicasPath` 指定定制资源内与 `scale.spec.replicas` 对应的 JSON 路径。

  - 此字段为必需值。
  - 只可以使用 `.spec` 下的 JSON 路径，只可使用带句点的路径。
  - 如果定制资源的 `specReplicasPath` 下没有取值，则针对 `/scale` 子资源执行 GET
    操作时会返回错误。

<!--
- `statusReplicasPath` defines the JSONPath inside of a custom resource that corresponds to `scale.status.replicas`.

  - It is a required value.
  - Only JSONPaths under `.status` and with the dot notation are allowed.
  - If there is no value under the `statusReplicasPath` in the custom resource,
the status replica value in the `/scale` subresource will default to 0.
-->
- `statusReplicasPath` 指定定制资源内与 `scale.status.replicas` 对应的 JSON 路径。

  - 此字段为必需值。
  - 只可以使用 `.status` 下的 JSON 路径，只可使用带句点的路径。
  - 如果定制资源的 `statusReplicasPath` 下没有取值，则针对 `/scale` 子资源的
    副本个数状态值默认为 0。

<!--
- `labelSelectorPath` defines the JSONPath inside of a custom resource that corresponds to `scale.status.selector`.

  - It is an optional value.
  - It must be set to work with HPA.
  - Only JSONPaths under `.status` or `.spec` and with the dot notation are allowed.
  - If there is no value under the `labelSelectorPath` in the custom resource,
the status selector value in the `/scale` subresource will default to the empty string.
  - The field pointed by this JSON path must be a string field (not a complex selector struct) which contains a serialized label selector in string form.
-->
- `labelSelectorPath` 指定定制资源内与 `scale.status.selector` 对应的 JSON 路径。

  - 此字段为可选值。
  - 此字段必须设置才能使用 HPA。
  - 只可以使用 `.status` 或 `.spec` 下的 JSON 路径，只可使用带句点的路径。
  - 如果定制资源的 `labelSelectorPath` 下没有取值，则针对 `/scale` 子资源的
    选择算符状态值默认为空字符串。
  - 此 JSON 路径所指向的字段必须是一个字符串字段（而不是复合的选择算符结构），
    其中包含标签选择算符串行化的字符串形式。

<!--
In the following example, both status and scale subresources are enabled.

Save the CustomResourceDefinition to `resourcedefinition.yaml`:
-->
在下面的例子中，`status` 和 `scale` 子资源都被启用。

将此 CustomResourceDefinition 保存到 `resourcedefinition.yaml` 文件：

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
            status:
              type: object
              properties:
                replicas:
                  type: integer
                labelSelector:
                  type: string
      # subresources 描述定制资源的子资源
      subresources:
        # status 启用 status 子资源
        status: {}
        # scale 启用 scale 子资源
        scale:
          # specReplicasPath 定义定制资源中对应 scale.spec.replicas 的 JSON 路径
          specReplicasPath: .spec.replicas
          # statusReplicasPath 定义定制资源中对应 scale.status.replicas 的 JSON 路径 
          statusReplicasPath: .status.replicas
          # labelSelectorPath  定义定制资源中对应 scale.status.selector 的 JSON 路径 
          labelSelectorPath: .status.labelSelector
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
    - ct
```

<!--
And create it:
-->
之后创建此 CustomResourceDefinition：

```shell
kubectl apply -f resourcedefinition.yaml
```

<!--
After the CustomResourceDefinition object has been created, you can create custom objects.

If you save the following YAML to `my-crontab.yaml`:
-->
CustomResourceDefinition 对象创建完毕之后，你可以创建定制对象，。

如果你将下面的 YAML 保存到 `my-crontab.yaml` 文件：

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
  replicas: 3
```

<!--
and create it:
-->
并创建定制对象：

```shell
kubectl apply -f my-crontab.yaml
```

<!--
Then new namespaced RESTful API endpoints are created at:
-->
那么会创建新的、命名空间作用域的 RESTful  API 端点：

```
/apis/stable.example.com/v1/namespaces/*/crontabs/status
```

<!-- and -->
和

```
/apis/stable.example.com/v1/namespaces/*/crontabs/scale
```

<!--
A custom resource can be scaled using the `kubectl scale` command.
For example, the following command sets `.spec.replicas` of the
custom resource created above to 5:
-->
定制资源可以使用 `kubectl scale` 命令来扩缩其规模。
例如下面的命令将前面创建的定制资源的 `.spec.replicas` 设置为 5：

```shell
kubectl scale --replicas=5 crontabs/my-new-cron-object
```
```
crontabs "my-new-cron-object" scaled
```
```shell
kubectl get crontabs my-new-cron-object -o jsonpath='{.spec.replicas}'
```
```
5
```

<!--
You can use a [PodDisruptionBudget](/docs/tasks/run-application/configure-pdb/) to protect custom
resources that have the scale subresource enabled.
-->
你可以使用 [PodDisruptionBudget](/zh-cn/docs/tasks/run-application/configure-pdb/)
来保护启用了 scale 子资源的定制资源。

<!--
### Categories
-->
### 分类   {#categories}

<!--
Categories is a list of grouped resources the custom resource belongs to (eg. `all`).
You can use `kubectl get <category-name>` to list the resources belonging to the category.

The following example adds `all` in the list of categories in the CustomResourceDefinition
and illustrates how to output the custom resource using `kubectl get all`.

Save the following CustomResourceDefinition to `resourcedefinition.yaml`:
-->
分类（Categories）是定制资源所归属的分组资源列表（例如，`all`）。
你可以使用 `kubectl get <分类名称>` 来列举属于某分类的所有资源。

下面的示例在 CustomResourceDefinition 中将 `all` 添加到分类列表中，
并展示了如何使用 `kubectl get all` 来输出定制资源：

将下面的 CustomResourceDefinition 保存到 `resourcedefinition.yaml` 文件中：

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: crontabs.stable.example.com
spec:
  group: stable.example.com
  versions:
    - name: v1
      served: true
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  scope: Namespaced
  names:
    plural: crontabs
    singular: crontab
    kind: CronTab
    shortNames:
    - ct
    # categories 是定制资源所归属的分类资源列表
    categories:
    - all
```

<!--
and create it:
-->
之后创建此 CRD：

```shell
kubectl apply -f resourcedefinition.yaml
```

<!--
After the CustomResourceDefinition object has been created, you can create custom objects.

Save the following YAML to `my-crontab.yaml`:
-->
创建了 CustomResourceDefinition 对象之后，你可以创建定制对象。

将下面的 YAML 保存到 `my-crontab.yaml` 中：

```yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
```

<!--
and create it:
-->
并创建定制对象：

```shell
kubectl apply -f my-crontab.yaml
```

<!--
You can specify the category when using `kubectl get`:
-->
你可以在使用 `kubectl get` 时指定分类：

```shell
kubectl get all
```

<!--
The output will include the custom resources of kind `CronTab`:
-->
输出中会包含类别为 `CronTab` 的定制资源：

```console
NAME                          AGE
crontabs/my-new-cron-object   3s
```

## {{% heading "whatsnext" %}}

<!--
* Read about [custom resources](/docs/concepts/extend-kubernetes/api-extension/custom-resources/).

* See [CustomResourceDefinition](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#customresourcedefinition-v1-apiextensions-k8s-io).

* Serve [multiple versions](/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definition-versioning/) of a
  CustomResourceDefinition.
-->
* 阅读了解[定制资源](/zh-cn/docs/concepts/extend-kubernetes/api-extension/custom-resources/)
* 参阅 [CustomResourceDefinition](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#customresourcedefinition-v1-apiextensions-k8s-io)
* 参阅支持 CustomResourceDefinition 的[多个版本](/zh-cn/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definition-versioning/)

