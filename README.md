# ansible-batch-ops-practice

## 项目简介

基于 Ansible 实现多主机自动化运维实践，包含主机初始化、配置文件批量分发、主机健康巡检三大能力，适合云原生运维方向校招项目展示。

> 运行环境：WSL2 Ubuntu
> 注意：WSL 内核裁剪，部分 sysctl 参数不支持，代码中做了异常兼容处理

## 目录结构

ansible-batch-ops-practice/
├── ansible.cfg        # ansible 本地配置
├── files/             # 待分发配置文件
├── inventory/
│   └── hosts.ini      # 主机清单
├── playbooks/
│   ├── init_env.yml
│   ├── file_dist.yml
│   └── health_check.yml
├── report/            # 自动生成的巡检报告输出目录
└── README.md

## ansible.cfg 全局配置

项目本地Ansible配置文件，统一指定：主机清单路径、默认远程用户、关闭主机key检查、关闭重试文件生成，简化本地实验调试。
> 注意：`host_key_checking=False` 仅用于学习实验环境，生产环境建议开启，保障SSH访问安全。
```ini
[defaults]
inventory = ./inventory/hosts.ini
remote_user = ubuntu
host_key_checking = False
retry_files_enabled = False

## 功能说明

1. **init_env.yml 环境初始化**

- 更新系统软件源，批量安装运维基础工具
- 部署 chrony，配置系统时间同步并设置开机自启
- 内核文件句柄调优；兼容 WSL，对不支持的内核参数配置忽略错误，保证流程不中断

2. **file_dist.yml 配置分发**

- 向目标主机推送 motd 登录提示文件
- 统一多主机登录提示，实现配置标准化

3. **health_check.yml 主机健康巡检**

- 采集主机 CPU、内存、系统版本、磁盘等基础信息
- 使用 `delegate_to: localhost`，在**控制端本机**创建目录、生成巡检报告，集中汇总多主机信息
- 报告自动输出至项目 `report/` 文件夹

## 使用方式

```
# 1. 主机初始化

ansible-playbook playbooks/init_env.yml -K

# 2. 批量分发配置文件

ansible-playbook playbooks/file_dist.yml -K

# 3. 主机健康巡检，生成巡检报告

ansible-playbook playbooks/health_check.yml -K
```

> 
> `-K`：手动输入 sudo 密码

## 项目踩坑 & 兼容说明（面试重点）

1. WSL2 精简内核，不存在 `fs.file-max`，直接执行 sysctl 模块会报错，添加 `ignore_errors: yes`，保障整条初始化流水线正常跑完
2. 巡检报告需要输出到控制机本地，默认 Ansible 在远端主机执行任务，增加 `delegate_to: localhost` 委托至本地执行文件创建、写入逻辑，解决路径与权限问题

## 项目截图

（在这里依次粘贴三张截图）

- init_env.yml 执行结果
- file_dist.yml 执行结果
- health_check.yml 执行结果

## 学习收获

熟悉 Ansible 常用模块（file、sysctl、copy、template、setup 等），理解 Ansible 远程执行模型、委托本地执行场景；体会批量运维、配置标准化、自动化巡检在运维工作中的价值。
