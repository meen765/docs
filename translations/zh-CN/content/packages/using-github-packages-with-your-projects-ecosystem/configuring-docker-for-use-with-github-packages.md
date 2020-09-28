---
title: 配置 Docker 用于 GitHub 包
intro: '您可以配置 Docker 客户端以使用 {{ site.data.variables.product.prodname_registry }} 发布和检索 docker 镜像。'
product: '{{ site.data.reusables.gated-features.packages }}'
redirect_from:
  - /articles/configuring-docker-for-use-with-github-package-registry
  - /github/managing-packages-with-github-package-registry/configuring-docker-for-use-with-github-package-registry
  - /github/managing-packages-with-github-packages/configuring-docker-for-use-with-github-packages
versions:
  free-pro-team: '*'
  enterprise-server: '>=2.22'
---

{{ site.data.reusables.package_registry.packages-ghes-release-stage }}

**注：**安装或发布 Docker 映像时，{{ site.data.variables.product.prodname_registry }} 当前不支持外部图层，如 Windows 映像。

### 向 {{ site.data.variables.product.prodname_registry }} 验证

{% warning %}

# Build the image with docker.pkg.github.com/&lt;em&gt;OWNER/REPOSITORY/IMAGE_NAME:VERSION&lt;/em&gt; # Assumes Dockerfile resides in the current working directory (.) $ docker build -t docker.pkg.github.com/octocat/octo-app/monalisa:1.0 . # Push the image to {{ site.data.variables.product.prodname_registry }} $ docker push docker.pkg.github.com/octocat/octo-app/monalisa:1.0

{% endwarning %}

您可以使用 `docker` 登录命令，通过 Docker 向 {{ site.data.variables.product.prodname_registry }} 验证。

{% if currentVersion != "free-pro-team@latest" %}

Before you can use the Docker registry on {{ site.data.variables.product.prodname_registry }}, the site administrator for {{ site.data.variables.product.product_location_enterprise }} must enable Docker support and subdomain isolation for your instance. For more information, see "[Managing GitHub Packages for your enterprise](/enterprise/admin/packages)."

{% endif %}

### 向 {{ site.data.variables.product.prodname_registry }} 验证

{{ site.data.reusables.package_registry.authenticate-packages }}

#### 使用个人访问令牌进行身份验证

{{ site.data.reusables.package_registry.required-scopes }}

您可以使用 `docker` 登录命令，通过 Docker 向 {{ site.data.variables.product.prodname_registry }} 验证。

为确保凭据安全，我们建议您将个人访问令牌保存在您计算机上的本地文件中，然后使用 Docker 的 `--password-stdin` 标志从本地文件读取您的令牌。

{% if currentVersion == "free-pro-team@latest" %}
{% raw %}
  ```shell
  $ cat <em>~/TOKEN.txt</em> | docker login https://docker.pkg.github.com -u <em>USERNAME</em> --password-stdin
  ```
{% endraw %}
{% endif %}

{% if currentVersion != "free-pro-team@latest" %}
{% raw %}
 ```shell
 $ cat <em>~/TOKEN.txt</em> | docker login docker.HOSTNAME -u <em>USERNAME</em> --password-stdin
```
{% endraw %}
{% endif %}

要使用此示例登录命令，请将 `USERNAME` 替换为您的 {{ site.data.variables.product.prodname_dotcom }} 用户名，将 `~/TOKEN.txt` 替换为您用于 {{ site.data.variables.product.prodname_dotcom }} 的个人访问令牌的文件路径。

更多信息请参阅“[Docker 登录](https://docs.docker.com/engine/reference/commandline/login/#provide-a-password-using-stdin)”。

#### 使用 `GITHUB_TOKEN` 进行身份验证

{{ site.data.reusables.package_registry.package-registry-with-github-tokens }}

### 发布包

{{ site.data.variables.product.prodname_registry }} 支持每个仓库的多个顶层 Docker 镜像。 仓库可以拥有任意数量的映像标记。 在发布或安装大于 10GB 的 Docker 映像（每个图层上限为 5GB）时，可能会遇到服务降级的情况。 更多信息请参阅 Docker 文档中的“[Docker 标记](https://docs.docker.com/engine/reference/commandline/tag/)”。

{{ site.data.reusables.package_registry.lowercase-name-field }}

{{ site.data.reusables.package_registry.viewing-packages }}

1. 使用 `docker images` 确定 docker 映像的名称和 ID。
  ```shell
  $ docker images
  > <&nbsp>
  > REPOSITORY        TAG        IMAGE ID       CREATED      SIZE
  > <em>IMAGE_NAME</em>        <em>VERSION</em>    <em>IMAGE_ID</em>       4 weeks ago  1.11MB
  ```
2. 使用 Docker 映像 ID 标记 docker 映像，将 *OWNER* 替换为拥有仓库的用户或组织帐户的名称，将 *REPOSITORY* 替换为包含项目的仓库的名称，将 *IMAGE_NAME* 替换为包或映像的名称，将 *VERSION* 替换为构建时的包版本。
{% if currentVersion != "free-pro-team@latest" %} *HOSTNAME* with the hostname of {{ site.data.variables.product.product_location_enterprise }},{% endif %} and *VERSION* with package version at build time.
  {% if currentVersion == "free-pro-team@latest" %}
  ```shell
  $ docker tag <em>IMAGE_ID</em> docker.pkg.github.com/<em>OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
  ```
  {% else %}
  ```shell
  如果尚未为包构建 docker 映像，请构建映像，将 <em x-id="3">OWNER</em> 替换为拥有仓库的用户或组织帐户的名称，将 <em x-id="3">REPOSITORY</em> 替换为包含项目的仓库的名称，将 <em x-id="3">IMAGE_NAME</em> 替换为包或映像的名称，将 <em x-id="3">VERSION</em> 替换为构建时的包版本，将 <em x-id="3">PATH</em> 替换为映像路径（如果映像未在当前工作目录中）。
  ```
  {% endif %}
3. 您可能首次发布新的 Docker 映像并将其命名为 `monalisa`。
{% if currentVersion != "free-pro-team@latest" %} *HOSTNAME* with the hostname of {{ site.data.variables.product.product_location_enterprise }},{% endif %} and *PATH* to the image if it isn't in the current working directory.s
  {% if currentVersion == "free-pro-team@latest" %}
  ```shell
  $ docker build -t docker.pkg.github.com/<em>OWNER/REPOSITORY/IMAGE_NAME:VERSION</em> <em>PATH</em>
  ```
  {% else %}
  ```shell
  $ docker build -t docker.<em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:VERSION</em> <em>PATH</em>
  ```
  {% endif %}
4. 将映像发布到 {{ site.data.variables.product.prodname_registry }}。
  {% if currentVersion == "free-pro-team@latest" %}
  ```shell
  $ docker push docker.pkg.github.com/<em>OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
  ```
  {% else %}
  ```shell
  $ docker push docker.<em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
  ```
  {% endif %}
  {% note %}

  **注：**必须使用 `IMAGE_NAME:VERSION` 推送映像，而不能使用 `IMAGE_NAME:SHA`。

  {% endnote %}

#### 发布 Docker 映像的示例

您可以使用映像 ID 将 `monalisa` 映像的 1.0 版本发布到 `octocat/octo-app` 仓库。

{% if currentVersion == "free-pro-team@latest" %}
```shell
$ docker images

> REPOSITORY           TAG      IMAGE ID      CREATED      SIZE
> monalisa             1.0      c75bebcdd211  4 weeks ago  1.11MB

# Tag the image with <em>OWNER/REPO/IMAGE_NAME</em>
$ docker tag c75bebcdd211 docker.pkg.github.com/octocat/octo-app/monalisa:1.0

# Push the image to {{ site.data.variables.product.prodname_registry }}
$ docker push docker.pkg.github.com/octocat/octo-app/monalisa:1.0
```

{% else %}

```shell
$ docker images

> REPOSITORY           TAG      IMAGE ID      CREATED      SIZE
> monalisa             1.0      c75bebcdd211  4 weeks ago  1.11MB

# Tag the image with <em>OWNER/REPO/IMAGE_NAME</em>
$ docker tag c75bebcdd211 docker.<em>HOSTNAME</em>/octocat/octo-app/monalisa:1.0

# Push the image to {{ site.data.variables.product.prodname_registry }}
$ docker push docker.<em>HOSTNAME</em>/octocat/octo-app/monalisa:1.0
```

{% endif %}

您可能首次发布新的 Docker 映像并将其命名为 `monalisa`。

{% if currentVersion == "free-pro-team@latest" %}
```shell
# Build the image with docker.pkg.github.com/<em>OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
# Assumes Dockerfile resides in the current working directory (.)
$ docker build -t docker.pkg.github.com/octocat/octo-app/monalisa:1.0 .

# Push the image to {{ site.data.variables.product.prodname_registry }}
$ docker push docker.pkg.github.com/octocat/octo-app/monalisa:1.0
```

{% else %}
```shell
# Build the image with docker.<em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:VERSION</em>
# Assumes Dockerfile resides in the current working directory (.)
$ docker build -t docker.<em>HOSTNAME</em>/octocat/octo-app/monalisa:1.0 .

# Push the image to {{ site.data.variables.product.prodname_registry }}
$ docker push docker.<em>HOSTNAME</em>/octocat/octo-app/monalisa:1.0
```
{% endif %}

### 安装包

您可以使用 `docker pull` 命令从 {{ site.data.variables.product.prodname_registry }} 安装 Docker 映像，将 *OWNER* 替换为拥有仓库的用户或组织帐户的名称，将 *REPOSITORY* 替换为包含项目的仓库的名称，将 *IMAGE_NAME* 替换为包或映像的名称，将 *TAG_NAME* 替换为要安装的映像的标记。 {{ site.data.reusables.package_registry.lowercase-name-field }}


{% if currentVersion == "free-pro-team@latest" %}
```shell
$ docker pull docker.pkg.github.com/<em>OWNER/REPOSITORY/IMAGE_NAME:TAG_NAME</em>
```
{% else %}
```shell
$ docker pull docker.<em>HOSTNAME/OWNER/REPOSITORY/IMAGE_NAME:TAG_NAME</em>
```
{% endif %}

{% note %}

**注：**必须使用 `IMAGE_NAME:VERSION` 推送映像，而不能使用 `IMAGE_NAME:SHA`。

{% endnote %}


### 延伸阅读

- “[删除包](/packages/publishing-and-managing-packages/deleting-a-package/)”