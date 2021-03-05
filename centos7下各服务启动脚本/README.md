**服务全部是yum安装后的启动脚本，如果是2进制安装需要根据安装目录和环境变量，按照实际需求修改**

**建议放在 /usr/lib/systemd/system/ 或者 /etc/systemd/system/ 下**

https://docs.gitlab.com/omnibus/settings/configuration.html



| 默认位置                                               | 权限   | 所有权           | 目的                 |
| :----------------------------------------------------- | :----- | :--------------- | :------------------- |
| `/var/opt/gitlab/git-data`                             | `0700` | `git`            | 存放仓库目录         |
| `/var/opt/gitlab/git-data/repositories`                | `2770` | `git:git`        | 拥有Git仓库          |
| `/var/opt/gitlab/gitlab-rails/shared`                  | `0751` | `git:gitlab-www` | 存放大对象目录       |
| `/var/opt/gitlab/gitlab-rails/shared/artifacts`        | `0700` | `git`            | 存放CI工件           |
| `/var/opt/gitlab/gitlab-rails/shared/external-diffs`   | `0700` | `git`            | 保留外部合并请求差异 |
| `/var/opt/gitlab/gitlab-rails/shared/lfs-objects`      | `0700` | `git`            | 存放LFS对象          |
| `/var/opt/gitlab/gitlab-rails/shared/packages`         | `0700` | `git`            | 存放软件包存储库     |
| `/var/opt/gitlab/gitlab-rails/shared/dependency_proxy` | `0700` | `git`            | 拥有依赖代理         |
| `/var/opt/gitlab/gitlab-rails/shared/terraform_state`  | `0700` | `git`            | 保持地形状态         |
| `/var/opt/gitlab/gitlab-rails/shared/pages`            | `0750` | `git:gitlab-www` | 存放用户页面         |
| `/var/opt/gitlab/gitlab-rails/uploads`                 | `0700` | `git`            | 存放用户附件         |
| `/var/opt/gitlab/gitlab-ci/builds`                     | `0700` | `git`            | 存放配置项构建日志   |
| `/var/opt/gitlab/.ssh`                                 | `0700` | `git:git`        | 持有授权钥匙         |

## 