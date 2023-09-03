# concourse-install
Concourse CI のインストール

## 参考
- ConcourseCI は v3.19以上のカーネルが必要。カーネル更新手順は以下を参考にする

    https://qiita.com/ryoctrl/items/8f786d3ccb2333ffc4d0


-  Concourse CI のインストール手順

    https://note.com/shift_tech/n/nead03c02b095


- Concourse CI のチュートリアル

    https://concoursetutorial-ja.site.lkj.io/


## 前提

- 以下の資料に従って harborをインストール済

    https://github.com/moriyamaES/harbor-install#readme


## 環境

```
# uname -a
Linux control-plane.minikube.internal 3.10.0-1160.el7.x86_64 #1 SMP Mon Oct 19 16:18:59 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```

```
# minikube version 
minikube version: v1.21.0
commit: 76d74191d82c47883dc7e1319ef7cebd3e00ee11
```

```
# docker version
Client: Docker Engine - Community
 Version:           20.10.7
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        f0df350
 Built:             Wed Jun  2 11:58:10 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.7
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       b0f5bc3
  Built:            Wed Jun  2 11:56:35 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.6
  GitCommit:        d71fcd7d8303cbf684402823e425e9dd2e99285d
 runc:
  Version:          1.0.0-rc95
  GitCommit:        b9ee9c6314599f1b4a7f497e1f1f856fe433d3b7
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

## カーネルアップデート

- ConcourseCI は v3.19以上のカーネルが必要。
- カーネルのバージョンをチェック

    ```
    # uname -r
    3.10.0-1160.el7.x86_64
    ```

    ```
    # rpm -qa | grep "^kernel" | sort
    kernel-3.10.0-1160.el7.x86_64
    kernel-tools-3.10.0-1160.el7.x86_64
    kernel-tools-libs-3.10.0-1160.el7.x86_64
    ```

- 3.10で要求を満たしていないのでカーネルをアップデートします。

- ELRepoを追加します。
- ELRepoにもバージョンがあるので[公式ページ](http://elrepo.org/tiki/tiki-index.php)を参考にしながらコマンドを実行していきます。
 ※ ここでは、[参考資料](https://qiita.com/ryoctrl/items/8f786d3ccb2333ffc4d0)の手順に従った


    ```
    # rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
    ```

    ```
    # rpm -Uvh https://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
    ```

    - 結果

        ```
        https://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm を取得中
        準備しています...              ################################# [100%]
        更新中 / インストール中...
        1:elrepo-release-7.0-3.el7.elrepo  ################################# [100%]
        ```

    ```
    # cat /etc/yum.repos.d/elrepo.repo
    ```

    - 結果

        ```
        ### Name: ELRepo.org Community Enterprise Linux Repository for el7
        ### URL: http://elrepo.org/

        [elrepo]
        name=ELRepo.org Community Enterprise Linux Repository - el7
        baseurl=http://elrepo.org/linux/elrepo/el7/$basearch/
                http://mirrors.coreix.net/elrepo/elrepo/el7/$basearch/
                http://mirror.rackspace.com/elrepo/elrepo/el7/$basearch/
                http://repos.lax-noc.com/elrepo/elrepo/el7/$basearch/
                http://mirror.ventraip.net.au/elrepo/elrepo/el7/$basearch/
        mirrorlist=http://mirrors.elrepo.org/mirrors-elrepo.el7
        enabled=1
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org
        protect=0

        [elrepo-testing]
        name=ELRepo.org Community Enterprise Linux Testing Repository - el7
        baseurl=http://elrepo.org/linux/testing/el7/$basearch/
                http://mirrors.coreix.net/elrepo/testing/el7/$basearch/
                http://mirror.rackspace.com/elrepo/testing/el7/$basearch/
                http://repos.lax-noc.com/elrepo/testing/el7/$basearch/
                http://mirror.ventraip.net.au/elrepo/testing/el7/$basearch/
        mirrorlist=http://mirrors.elrepo.org/mirrors-elrepo-testing.el7
        enabled=0
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org
        protect=0

        [elrepo-kernel]
        name=ELRepo.org Community Enterprise Linux Kernel Repository - el7
        baseurl=http://elrepo.org/linux/kernel/el7/$basearch/
                http://mirrors.coreix.net/elrepo/kernel/el7/$basearch/
                http://mirror.rackspace.com/elrepo/kernel/el7/$basearch/
                http://repos.lax-noc.com/elrepo/kernel/el7/$basearch/
                http://mirror.ventraip.net.au/elrepo/kernel/el7/$basearch/
        mirrorlist=http://mirrors.elrepo.org/mirrors-elrepo-kernel.el7
        enabled=0
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org
        protect=0

        [elrepo-extras]
        name=ELRepo.org Community Enterprise Linux Extras Repository - el7
        baseurl=http://elrepo.org/linux/extras/el7/$basearch/
                http://mirrors.coreix.net/elrepo/extras/el7/$basearch/
                http://mirror.rackspace.com/elrepo/extras/el7/$basearch/
                http://repos.lax-noc.com/elrepo/extras/el7/$basearch/
                http://mirror.ventraip.net.au/elrepo/extras/el7/$basearch/
        mirrorlist=http://mirrors.elrepo.org/mirrors-elrepo-extras.el7
        enabled=0
        gpgcheck=1
        gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org
        protect=0
        ```
- ELRepoが追加出来たら、以下を実行

    ```
    # yum --enablerepo=elrepo-kernel -y install kernel-ml
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        Determining fastest mirrors
        * base: ftp-srv2.kddilabs.jp
        * elrepo: ftp.yz.yamagata-u.ac.jp
        * elrepo-kernel: ftp.yz.yamagata-u.ac.jp
        * extras: ftp-srv2.kddilabs.jp
        * updates: ftp-srv2.kddilabs.jp
        base                                                                               | 3.6 kB  00:00:00     
        docker-ce-stable                                                                   | 3.5 kB  00:00:00     
        elrepo                                                                             | 3.0 kB  00:00:00     
        elrepo-kernel                                                                      | 3.0 kB  00:00:00     
        extras                                                                             | 2.9 kB  00:00:00     
        ius                                                                                | 1.3 kB  00:00:00     
        updates                                                                            | 2.9 kB  00:00:00     
        (1/3): elrepo/primary_db                                                           | 356 kB  00:00:00     
        (2/3): updates/7/x86_64/primary_db                                                 |  22 MB  00:00:00     
        (3/3): elrepo-kernel/primary_db                                                    | 2.1 MB  00:00:01     
        依存性の解決をしています
        --> トランザクションの確認を実行しています。
        ---> パッケージ kernel-ml.x86_64 0:6.4.12-1.el7.elrepo を インストール
        --> 依存性解決を終了しました。

        依存性を解決しました

        ==========================================================================================================
        Package               アーキテクチャー   バージョン                      リポジトリー               容量
        ==========================================================================================================
        インストール中:
        kernel-ml             x86_64             6.4.12-1.el7.elrepo             elrepo-kernel              66 M

        トランザクションの要約
        ==========================================================================================================
        インストール  1 パッケージ

        総ダウンロード容量: 66 M
        インストール容量: 339 M
        Downloading packages:
        kernel-ml-6.4.12-1.el7.elrepo.x86_64.rpm                                           |  66 MB  00:00:07     
        Running transaction check
        Running transaction test
        Transaction test succeeded
        Running transaction
        警告: RPMDB は yum 以外で変更されました。
        インストール中          : kernel-ml-6.4.12-1.el7.elrepo.x86_64                                      1/1 
        検証中                  : kernel-ml-6.4.12-1.el7.elrepo.x86_64                                      1/1 

        インストール:
        kernel-ml.x86_64 0:6.4.12-1.el7.elrepo                                                                  

        完了しました!
        ```

- 更新内容を確認

    ```
    # rpm -qa | grep "^kernel" | sort
    ```

    - 結果
    - kernel-mlが更新されていればOKです。

        ```
        kernel-3.10.0-1160.el7.x86_64
        kernel-ml-6.4.12-1.el7.elrepo.x86_64
        kernel-tools-3.10.0-1160.el7.x86_64
        kernel-tools-libs-3.10.0-1160.el7.x86_64
        ```

- このままだと次回の再起動時は元の古いカーネルで起動されてしまうため起動設定を変えます。

    ```
    # grub2-editenv list
    ```

    - 結果

        ```
        saved_entry=CentOS Linux (3.10.0-1160.el7.x86_64) 7 (Core)
        ```

    ```
    # awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
    ```

    - 結果

        ```
        0 : CentOS Linux (6.4.12-1.el7.elrepo.x86_64) 7 (Core)
        1 : CentOS Linux (3.10.0-1160.el7.x86_64) 7 (Core)
        2 : CentOS Linux (0-rescue-b89cc28406e54592a5591681ba669192) 7 (Core)
        ```

- 今回更新した6.4.10で起動したいので0をsetします。

    ```
    # grub2-set-default 0
    ```

- カーネルヘッダやツールを一つずつ入れ替えていきます

    ```
    # yum --enablerepo=elrepo-kernel -y swap kernel-headers -- kernel-ml-headers
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        引数に一致しません: kernel-headers
        swap remove kernel-headers
        ```
    
    ```
    # yum --enablerepo=elrepo-kernel -y swap kernel-tools-libs -- kernel-ml-tools-libs
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        Loading mirror speeds from cached hostfile
        * base: ftp-srv2.kddilabs.jp
        * elrepo: ftp.yz.yamagata-u.ac.jp
        * elrepo-kernel: ftp.yz.yamagata-u.ac.jp
        * extras: ftp-srv2.kddilabs.jp
        * updates: ftp-srv2.kddilabs.jp
        依存性の解決をしています
        --> トランザクションの確認を実行しています。
        ---> パッケージ kernel-ml-tools-libs.x86_64 0:6.4.12-1.el7.elrepo を インストール
        ---> パッケージ kernel-tools-libs.x86_64 0:3.10.0-1160.el7 を 削除
        --> 依存性の処理をしています: kernel-tools-libs = 3.10.0-1160.el7 のパッケージ: kernel-tools-3.10.0-1160.el7.x86_64
        --> トランザクションの確認を実行しています。
        ---> パッケージ kernel-tools.x86_64 0:3.10.0-1160.el7 を 削除
        --> 依存性解決を終了しました。

        依存性を解決しました

        ==========================================================================================================
        Package                       アーキテクチャー
                                                    バージョン                    リポジトリー            容量
        ==========================================================================================================
        インストール中:
        kernel-ml-tools-libs          x86_64          6.4.12-1.el7.elrepo           elrepo-kernel          183 k
        削除中:
        kernel-tools-libs             x86_64          3.10.0-1160.el7               @anaconda               18 k
        依存性関連での削除をします:
        kernel-tools                  x86_64          3.10.0-1160.el7               @anaconda              337 k

        トランザクションの要約
        ==========================================================================================================
        インストール  1 パッケージ
        削除          1 パッケージ (+1 個の依存関係のパッケージ)

        総ダウンロード容量: 183 k
        Downloading packages:
        kernel-ml-tools-libs-6.4.12-1.el7.elrepo.x86_64.rpm                                | 183 kB  00:00:00     
        Running transaction check
        Running transaction test
        Transaction test succeeded
        Running transaction
        インストール中          : kernel-ml-tools-libs-6.4.12-1.el7.elrepo.x86_64                           1/3 
        削除中                  : kernel-tools-3.10.0-1160.el7.x86_64                                       2/3 
        削除中                  : kernel-tools-libs-3.10.0-1160.el7.x86_64                                  3/3 
        検証中                  : kernel-ml-tools-libs-6.4.12-1.el7.elrepo.x86_64                           1/3 
        検証中                  : kernel-tools-3.10.0-1160.el7.x86_64                                       2/3 
        検証中                  : kernel-tools-libs-3.10.0-1160.el7.x86_64                                  3/3 

        削除しました:
        kernel-tools-libs.x86_64 0:3.10.0-1160.el7                                                              

        依存性の削除をしました:
        kernel-tools.x86_64 0:3.10.0-1160.el7                                                                   

        インストール:
        kernel-ml-tools-libs.x86_64 0:6.4.12-1.el7.elrepo                                                       

        完了しました!
        ```

    ```
    # yum --enablerepo=elrepo-kernel -y install kernel-ml-tools
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        Loading mirror speeds from cached hostfile
        * base: ftp-srv2.kddilabs.jp
        * elrepo: ftp.yz.yamagata-u.ac.jp
        * elrepo-kernel: ftp.yz.yamagata-u.ac.jp
        * extras: ftp-srv2.kddilabs.jp
        * updates: ftp-srv2.kddilabs.jp
        依存性の解決をしています
        --> トランザクションの確認を実行しています。
        ---> パッケージ kernel-ml-tools.x86_64 0:6.4.12-1.el7.elrepo を インストール
        --> 依存性解決を終了しました。

        依存性を解決しました

        ==========================================================================================================
        Package                   アーキテクチャー バージョン                      リポジトリー             容量
        ==========================================================================================================
        インストール中:
        kernel-ml-tools           x86_64           6.4.12-1.el7.elrepo             elrepo-kernel           296 k

        トランザクションの要約
        ==========================================================================================================
        インストール  1 パッケージ

        総ダウンロード容量: 296 k
        インストール容量: 435 k
        Downloading packages:
        kernel-ml-tools-6.4.12-1.el7.elrepo.x86_64.rpm                                     | 296 kB  00:00:00     
        Running transaction check
        Running transaction test
        Transaction test succeeded
        Running transaction
        インストール中          : kernel-ml-tools-6.4.12-1.el7.elrepo.x86_64                                1/1 
        検証中                  : kernel-ml-tools-6.4.12-1.el7.elrepo.x86_64                                1/1 

        インストール:
        kernel-ml-tools.x86_64 0:6.4.12-1.el7.elrepo                                                            
        ```

    ```
    # yum --enablerepo=elrepo-kernel -y swap kernel-devel -- kernel-ml-devel
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        引数に一致しません: kernel-devel
        swap remove kernel-devel
        ```

- 最後に古いカーネルを削除します。

    ```
    # yum -y remove kernel
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        実行中のカーネルを飛ばします: kernel-3.10.0-1160.el7.x86_64
        削除対象とマークされたパッケージはありません
        ```

    ```
    # ls -l /lib/modules
    ```

    - 結果

        ```
        計 8
        drwxr-xr-x. 7 root root 4096  7月  2 06:25 3.10.0-1160.el7.x86_64
        drwxr-xr-x. 7 root root 4096  8月 31 12:34 6.4.12-1.el7.elrepo.x86_64
        ```
    
- 自動起動が新しいカーネルになっていることを確認して再起動

    ```
    # grub2-editenv list
    ```
    - 結果

        ```
        saved_entry=0
        ```

    ```
     # shutdown -r now
    ```
    
- 再起動後、カーネルがバージョンアップされていることを確認


    ```
    # uname -r
    ```

    - 結果

        ```
        6.4.12-1.el7.elrepo.x86_64
        ```

## minikube が起動できないことを確認

- minikube の起動を確認

    ```
    # minikube start --vm-driver=none
    ```

    - 結果

        ```
        😄  Centos 7.9.2009 上の minikube v1.21.0
        ✨  プロフィールを元に、 none ドライバを使用します

        🧯  The requested memory allocation of 2200MiB does not leave room for system overhead (total system memory: 2909MiB). You may face stability issues.
        💡  提案: Start minikube with less memory allocated: 'minikube start --memory=2200mb'

        👍  コントロールプレーンのノード minikube を minikube 上で起動しています
        🎉  minikube 1.31.2 が利用可能です! 以下のURLでダウンロードできます。 https://github.com/kubernetes/minikube/releases/tag/v1.31.2
        💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

        🔄  既存の none bare metal machine を "minikube" のために再起動しています...
        ℹ️  OS は CentOS Linux 7 (Core) です。
        🐳  Docker 20.10.7 で Kubernetes v1.20.7 を準備しています...
        ❌  Problems detected in kubelet:
            8月 31 13:10:31 control-plane.minikube.internal kubelet[14227]: F0831 13:10:31.548306   14227 kubelet.go:1367] Failed to start ContainerManager failed to get rootfs info: unable to find data in memory cache
        🤦  Unable to restart cluster, will reset it: apiserver healthz: apiserver process never appeared
            ▪ 証明書と鍵を作成しています...
            ▪ Control Plane を起動しています...
        💢  初期化が失敗しました。再施行します。 wait: /bin/bash -c "sudo env PATH=/var/lib/minikube/binaries/v1.20.7:$PATH kubeadm init --config /var/tmp/minikube/kubeadm.yaml  --ignore-preflight-errors=DirAvailable--etc-kubernetes-manifests,DirAvailable--var-lib-minikube,DirAvailable--var-lib-minikube-etcd,FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml,FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml,FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml,FileAvailable--etc-kubernetes-manifests-etcd.yaml,Port-10250,Swap,Mem": exit status 1
        stdout:
        [init] Using Kubernetes version: v1.20.7
        [preflight] Running pre-flight checks
        [preflight] Pulling images required for setting up a Kubernetes cluster
        [preflight] This might take a minute or two, depending on the speed of your internet connection
        [preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
        [certs] Using certificateDir folder "/var/lib/minikube/certs"
        [certs] Using existing ca certificate authority
        [certs] Using existing apiserver certificate and key on disk
        [certs] Using existing apiserver-kubelet-client certificate and key on disk
        [certs] Using existing front-proxy-ca certificate authority
        [certs] Using existing front-proxy-client certificate and key on disk
        [certs] Using existing etcd/ca certificate authority
        [certs] Using existing etcd/server certificate and key on disk
        [certs] Using existing etcd/peer certificate and key on disk
        [certs] Using existing etcd/healthcheck-client certificate and key on disk
        [certs] Using existing apiserver-etcd-client certificate and key on disk
        [certs] Using the existing "sa" key
        [kubeconfig] Using kubeconfig folder "/etc/kubernetes"
        [kubeconfig] Writing "admin.conf" kubeconfig file
        [kubeconfig] Writing "kubelet.conf" kubeconfig file
        [kubeconfig] Writing "controller-manager.conf" kubeconfig file
        [kubeconfig] Writing "scheduler.conf" kubeconfig file
        [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
        [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
        [kubelet-start] Starting the kubelet
        [control-plane] Using manifest folder "/etc/kubernetes/manifests"
        [control-plane] Creating static Pod manifest for "kube-apiserver"
        [control-plane] Creating static Pod manifest for "kube-controller-manager"
        [control-plane] Creating static Pod manifest for "kube-scheduler"
        [etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
        [wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
        [kubelet-check] Initial timeout of 40s passed.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.

                Unfortunately, an error has occurred:
                        timed out waiting for the condition

                This error is likely caused by:
                        - The kubelet is not running
                        - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

                If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
                        - 'systemctl status kubelet'
                        - 'journalctl -xeu kubelet'

                Additionally, a control plane component may have crashed or exited when started by the container runtime.
                To troubleshoot, list all containers using your preferred container runtimes CLI.

                Here is one example how you may list all Kubernetes containers running in docker:
                        - 'docker ps -a | grep kube | grep -v pause'
                        Once you have found the failing container, you can inspect its logs with:
                        - 'docker logs CONTAINERID'


        stderr:
                [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
                [WARNING Swap]: running with swap on is not supported. Please disable swap
                [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.7. Latest validated version: 19.03
                [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
        error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
        To see the stack trace of this error execute with --v=5 or higher

            ▪ 証明書と鍵を作成しています...
            ▪ Control Plane を起動しています...

        💣  クラスタを起動中にエラーが発生しました: wait: /bin/bash -c "sudo env PATH=/var/lib/minikube/binaries/v1.20.7:$PATH kubeadm init --config /var/tmp/minikube/kubeadm.yaml  --ignore-preflight-errors=DirAvailable--etc-kubernetes-manifests,DirAvailable--var-lib-minikube,DirAvailable--var-lib-minikube-etcd,FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml,FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml,FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml,FileAvailable--etc-kubernetes-manifests-etcd.yaml,Port-10250,Swap,Mem": exit status 1
        stdout:
        [init] Using Kubernetes version: v1.20.7
        [preflight] Running pre-flight checks
        [preflight] Pulling images required for setting up a Kubernetes cluster
        [preflight] This might take a minute or two, depending on the speed of your internet connection
        [preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
        [certs] Using certificateDir folder "/var/lib/minikube/certs"
        [certs] Using existing ca certificate authority
        [certs] Using existing apiserver certificate and key on disk
        [certs] Using existing apiserver-kubelet-client certificate and key on disk
        [certs] Using existing front-proxy-ca certificate authority
        [certs] Using existing front-proxy-client certificate and key on disk
        [certs] Using existing etcd/ca certificate authority
        [certs] Using existing etcd/server certificate and key on disk
        [certs] Using existing etcd/peer certificate and key on disk
        [certs] Using existing etcd/healthcheck-client certificate and key on disk
        [certs] Using existing apiserver-etcd-client certificate and key on disk
        [certs] Using the existing "sa" key
        [kubeconfig] Using kubeconfig folder "/etc/kubernetes"
        [kubeconfig] Writing "admin.conf" kubeconfig file
        [kubeconfig] Writing "kubelet.conf" kubeconfig file
        [kubeconfig] Writing "controller-manager.conf" kubeconfig file
        [kubeconfig] Writing "scheduler.conf" kubeconfig file
        [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
        [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
        [kubelet-start] Starting the kubelet
        [control-plane] Using manifest folder "/etc/kubernetes/manifests"
        [control-plane] Creating static Pod manifest for "kube-apiserver"
        [control-plane] Creating static Pod manifest for "kube-controller-manager"
        [control-plane] Creating static Pod manifest for "kube-scheduler"
        [etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
        [wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
        [kubelet-check] Initial timeout of 40s passed.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.

                Unfortunately, an error has occurred:
                        timed out waiting for the condition

                This error is likely caused by:
                        - The kubelet is not running
                        - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

                If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
                        - 'systemctl status kubelet'
                        - 'journalctl -xeu kubelet'

                Additionally, a control plane component may have crashed or exited when started by the container runtime.
                To troubleshoot, list all containers using your preferred container runtimes CLI.

                Here is one example how you may list all Kubernetes containers running in docker:
                        - 'docker ps -a | grep kube | grep -v pause'
                        Once you have found the failing container, you can inspect its logs with:
                        - 'docker logs CONTAINERID'


        stderr:
                [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
                [WARNING Swap]: running with swap on is not supported. Please disable swap
                [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.7. Latest validated version: 19.03
                [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
        error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
        To see the stack trace of this error execute with --v=5 or higher


        ╭────────────────────────────────────────────────────────────────────╮
        │                                                                    │
        │    😿  If the above advice does not help, please let us know:      │
        │    👉  https://github.com/kubernetes/minikube/issues/new/choose    │
        │                                                                    │
        │    Please attach the following file to the GitHub issue:           │
        │    - /root/.minikube/logs/lastStart.txt                            │
        │                                                                    │
        ╰────────────────────────────────────────────────────────────────────╯


        ❌  Exiting due to K8S_KUBELET_NOT_RUNNING: wait: /bin/bash -c "sudo env PATH=/var/lib/minikube/binaries/v1.20.7:$PATH kubeadm init --config /var/tmp/minikube/kubeadm.yaml  --ignore-preflight-errors=DirAvailable--etc-kubernetes-manifests,DirAvailable--var-lib-minikube,DirAvailable--var-lib-minikube-etcd,FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml,FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml,FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml,FileAvailable--etc-kubernetes-manifests-etcd.yaml,Port-10250,Swap,Mem": exit status 1
        stdout:
        [init] Using Kubernetes version: v1.20.7
        [preflight] Running pre-flight checks
        [preflight] Pulling images required for setting up a Kubernetes cluster
        [preflight] This might take a minute or two, depending on the speed of your internet connection
        [preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
        [certs] Using certificateDir folder "/var/lib/minikube/certs"
        [certs] Using existing ca certificate authority
        [certs] Using existing apiserver certificate and key on disk
        [certs] Using existing apiserver-kubelet-client certificate and key on disk
        [certs] Using existing front-proxy-ca certificate authority
        [certs] Using existing front-proxy-client certificate and key on disk
        [certs] Using existing etcd/ca certificate authority
        [certs] Using existing etcd/server certificate and key on disk
        [certs] Using existing etcd/peer certificate and key on disk
        [certs] Using existing etcd/healthcheck-client certificate and key on disk
        [certs] Using existing apiserver-etcd-client certificate and key on disk
        [certs] Using the existing "sa" key
        [kubeconfig] Using kubeconfig folder "/etc/kubernetes"
        [kubeconfig] Writing "admin.conf" kubeconfig file
        [kubeconfig] Writing "kubelet.conf" kubeconfig file
        [kubeconfig] Writing "controller-manager.conf" kubeconfig file
        [kubeconfig] Writing "scheduler.conf" kubeconfig file
        [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
        [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
        [kubelet-start] Starting the kubelet
        [control-plane] Using manifest folder "/etc/kubernetes/manifests"
        [control-plane] Creating static Pod manifest for "kube-apiserver"
        [control-plane] Creating static Pod manifest for "kube-controller-manager"
        [control-plane] Creating static Pod manifest for "kube-scheduler"
        [etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
        [wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
        [kubelet-check] Initial timeout of 40s passed.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
        [kubelet-check] It seems like the kubelet isn't running or healthy.
        [kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.

                Unfortunately, an error has occurred:
                        timed out waiting for the condition

                This error is likely caused by:
                        - The kubelet is not running
                        - The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

                If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
                        - 'systemctl status kubelet'
                        - 'journalctl -xeu kubelet'

                Additionally, a control plane component may have crashed or exited when started by the container runtime.
                To troubleshoot, list all containers using your preferred container runtimes CLI.

                Here is one example how you may list all Kubernetes containers running in docker:
                        - 'docker ps -a | grep kube | grep -v pause'
                        Once you have found the failing container, you can inspect its logs with:
                        - 'docker logs CONTAINERID'


        stderr:
                [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
                [WARNING Swap]: running with swap on is not supported. Please disable swap
                [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.7. Latest validated version: 19.03
                [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
        error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
        To see the stack trace of this error execute with --v=5 or higher

        💡  提案: Check output of 'journalctl -xeu kubelet', try passing --extra-config=kubelet.cgroup-driver=systemd to minikube start
        🍿  Related issue: https://github.com/kubernetes/minikube/issues/4172
        ```

- minikube の起動が起動できなくなった

## Dockerをバージョンアップする

- 現在の環境は、Dockerをyumでインストールした。

    ```
    yum install -y \
    docker-ce-20.10.7 \
    docker-ce-cli-20.10.7 \
    containerd.io-1.4.6
    ```

- 上記を考慮して、Dockerをバージョンアップする

- インストールされているDockerのバージョンを確認する。

    ```
    # yum list installed | grep -e "docker" -e "container"
    ```

    - 結果

        ```
        container-selinux.noarch                    2:2.119.2-1.911c772.el7_8  @extras  
        containerd.io.x86_64                        1.4.6-3.1.el7              @docker-ce-stable
        docker-ce.x86_64                            3:20.10.7-3.el7            @docker-ce-stable
        docker-ce-cli.x86_64                        1:20.10.7-3.el7            @docker-ce-stable
        docker-ce-rootless-extras.x86_64            24.0.4-1.el7               @docker-ce-stable
        docker-scan-plugin.x86_64                   0.23.0-3.el7               @docker-ce-stable
        ```

- インストールしたパッケージをアンインストール

    ```
    # yum remove docker-ce.x86_64
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        依存性の解決をしています
        --> トランザクションの確認を実行しています。
        ---> パッケージ docker-ce.x86_64 3:20.10.7-3.el7 を 削除
        --> 依存性の処理をしています: docker-ce のパッケージ: docker-ce-rootless-extras-24.0.4-1.el7.x86_64
        --> トランザクションの確認を実行しています。
        ---> パッケージ docker-ce-rootless-extras.x86_64 0:24.0.4-1.el7 を 削除
        --> 依存性解決を終了しました。

        依存性を解決しました

        ================================================================================================
        Package                        アーキテクチャー
                                                    バージョン             リポジトリー            容量
        ================================================================================================
        削除中:
        docker-ce                      x86_64      3:20.10.7-3.el7        @docker-ce-stable      115 M
        依存性関連での削除をします:
        docker-ce-rootless-extras      x86_64      24.0.4-1.el7           @docker-ce-stable       19 M

        トランザクションの要約
        ================================================================================================
        削除  1 パッケージ (+1 個の依存関係のパッケージ)

        インストール容量: 135 M
        上記の処理を行います。よろしいでしょうか？ [y/N]y
        Downloading packages:
        Running transaction check
        Running transaction test
        Transaction test succeeded
        Running transaction
        削除中                  : docker-ce-rootless-extras-24.0.4-1.el7.x86_64                   1/2 
        削除中                  : 3:docker-ce-20.10.7-3.el7.x86_64                                2/2 
        検証中                  : 3:docker-ce-20.10.7-3.el7.x86_64                                1/2 
        検証中                  : docker-ce-rootless-extras-24.0.4-1.el7.x86_64                   2/2 

        削除しました:
        docker-ce.x86_64 3:20.10.7-3.el7                                                              

        依存性の削除をしました:
        docker-ce-rootless-extras.x86_64 0:24.0.4-1.el7                                               

        完了しました!
        ```
    
    ```
    # yum remove docker-ce-cli.x86_64
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        依存性の解決をしています
        --> トランザクションの確認を実行しています。
        ---> パッケージ docker-ce-cli.x86_64 1:20.10.7-3.el7 を 削除
        --> 依存性解決を終了しました。

        依存性を解決しました

        ================================================================================================
        Package               アーキテクチャー
                                            バージョン                リポジトリー               容量
        ================================================================================================
        削除中:
        docker-ce-cli         x86_64         1:20.10.7-3.el7           @docker-ce-stable         156 M

        トランザクションの要約
        ================================================================================================
        削除  1 パッケージ

        インストール容量: 156 M
        上記の処理を行います。よろしいでしょうか？ [y/N]y
        Downloading packages:
        Running transaction check
        Running transaction test
        Transaction test succeeded
        Running transaction
        削除中                  : 1:docker-ce-cli-20.10.7-3.el7.x86_64                            1/1 
        検証中                  : 1:docker-ce-cli-20.10.7-3.el7.x86_64                            1/1 

        削除しました:
        docker-ce-cli.x86_64 1:20.10.7-3.el7                                                          

        完了しました!
        ```

    ```
    # yum remove containerd.io.x86_64
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        依存性の解決をしています
        --> トランザクションの確認を実行しています。
        ---> パッケージ containerd.io.x86_64 0:1.4.6-3.1.el7 を 削除
        --> 依存性解決を終了しました。

        依存性を解決しました

        ================================================================================================
        Package                アーキテクチャー
                                                バージョン             リポジトリー                容量
        ================================================================================================
        削除中:
        containerd.io          x86_64          1.4.6-3.1.el7          @docker-ce-stable          129 M

        トランザクションの要約
        ================================================================================================
        削除  1 パッケージ

        インストール容量: 129 M
        上記の処理を行います。よろしいでしょうか？ [y/N]y
        Downloading packages:
        Running transaction check
        Running transaction test
        Transaction test succeeded
        Running transaction
        削除中                  : containerd.io-1.4.6-3.1.el7.x86_64                              1/1 
        検証中                  : containerd.io-1.4.6-3.1.el7.x86_64                              1/1 

        削除しました:
        containerd.io.x86_64 0:1.4.6-3.1.el7                                                          

        完了しました!
        ```
    
- 不要な依存関係や設定ファイルが残っている可能性があるため、アンインストール後に不要なファイルや設定を削除する

    ```
    # yum autoremove
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        依存性の解決をしています
        --> トランザクションの確認を実行しています。
        ---> パッケージ container-selinux.noarch 2:2.119.2-1.911c772.el7_8 を 削除
        ---> パッケージ docker-scan-plugin.x86_64 0:0.23.0-3.el7 を 削除
        ---> パッケージ fuse-overlayfs.x86_64 0:0.7.2-6.el7_8 を 削除
        ---> パッケージ slirp4netns.x86_64 0:0.4.3-4.el7_8 を 削除
        --> 依存性解決を終了しました。
        --> Finding unneeded leftover dependencies
        ---> Marking fuse3-libs to be removed - no longer needed by fuse-overlayfs
        Found and removing 1 unneeded dependencies
        --> トランザクションの確認を実行しています。
        ---> パッケージ fuse3-libs.x86_64 0:3.6.1-4.el7 を 削除
        --> 依存性解決を終了しました。

        依存性を解決しました

        ================================================================================================
        Package                アーキテクチャー
                                        バージョン                       リポジトリー           容量
        ================================================================================================
        削除中:
        container-selinux      noarch     2:2.119.2-1.911c772.el7_8        @extras                41 k
        docker-scan-plugin     x86_64     0.23.0-3.el7                     @docker-ce-stable      12 M
        fuse-overlayfs         x86_64     0.7.2-6.el7_8                    @extras               116 k
        slirp4netns            x86_64     0.4.3-4.el7_8                    @extras               169 k
        依存性関連での削除をします:
        fuse3-libs             x86_64     3.6.1-4.el7                      @extras               270 k

        トランザクションの要約
        ================================================================================================
        削除  4 パッケージ (+1 個の依存関係のパッケージ)

        インストール容量: 13 M
        上記の処理を行います。よろしいでしょうか？ [y/N]y
        Downloading packages:
        Running transaction check
        Running transaction test
        Transaction test succeeded
        Running transaction
        削除中                  : fuse-overlayfs-0.7.2-6.el7_8.x86_64                             1/5 
        削除中                  : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch              2/5 
        削除中                  : docker-scan-plugin-0.23.0-3.el7.x86_64                          3/5 
        削除中                  : fuse3-libs-3.6.1-4.el7.x86_64                                   4/5 
        削除中                  : slirp4netns-0.4.3-4.el7_8.x86_64                                5/5 
        検証中                  : fuse3-libs-3.6.1-4.el7.x86_64                                   1/5 
        検証中                  : fuse-overlayfs-0.7.2-6.el7_8.x86_64                             2/5 
        検証中                  : docker-scan-plugin-0.23.0-3.el7.x86_64                          3/5 
        検証中                  : slirp4netns-0.4.3-4.el7_8.x86_64                                4/5 
        検証中                  : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch              5/5 

        削除しました:
        container-selinux.noarch 2:2.119.2-1.911c772.el7_8  docker-scan-plugin.x86_64 0:0.23.0-3.el7 
        fuse-overlayfs.x86_64 0:0.7.2-6.el7_8               slirp4netns.x86_64 0:0.4.3-4.el7_8       

        依存性の削除をしました:
        fuse3-libs.x86_64 0:3.6.1-4.el7                                                               

        完了しました!
        ```

- アンインストールを確認する。

    ```
    # yum list installed | grep -e "docker" -e "container"
    ```

    - 結果 → 表示なし（正常）

        ```
        ```

- インストールするソフトの最新をバージョンを確認する

    ```
    # yum --showduplicates list | grep \
    -e "^docker-ce.x86_64" \
    -e "^docker-ce-cli.x86_64" \
    -e "^containerd.io.x86_64"
    ```

    -  結果、それぞれの最新バージョンは以下と判明

        ```
        docker-ce.x86_64                         3:24.0.5-1.el7                docker-ce-stable
        ```

        ```
        docker-ce-cli.x86_64                     1:24.0.5-1.el7                docker-ce-stable
        ```

        ```
        containerd.io.x86_64                     1.6.22-3.1.el7                docker-ce-stable
        ```

- 上記結果から、Dockerの最新版のインストールコマンドは以下となる

    ```
    # yum install -y \
    docker-ce-24.0.5 \
    docker-ce-cli-24.0.5 \
    containerd.io-1.6.22
    ```

    - 結果

        ```
        読み込んだプラグイン:fastestmirror, langpacks
        Loading mirror speeds from cached hostfile
        * base: ftp-srv2.kddilabs.jp
        * elrepo: ftp.yz.yamagata-u.ac.jp
        * extras: ftp-srv2.kddilabs.jp
        * updates: ftp-srv2.kddilabs.jp
        依存性の解決をしています
        --> トランザクションの確認を実行しています。
        ---> パッケージ containerd.io.x86_64 0:1.6.22-3.1.el7 を インストール
        --> 依存性の処理をしています: container-selinux >= 2:2.74 のパッケージ: containerd.io-1.6.22-3.1.el7.x86_64
        ---> パッケージ docker-ce.x86_64 3:24.0.5-1.el7 を インストール
        --> 依存性の処理をしています: docker-ce-rootless-extras のパッケージ: 3:docker-ce-24.0.5-1.el7.x86_64
        ---> パッケージ docker-ce-cli.x86_64 1:24.0.5-1.el7 を インストール
        --> 依存性の処理をしています: docker-buildx-plugin のパッケージ: 1:docker-ce-cli-24.0.5-1.el7.x86_64
        --> 依存性の処理をしています: docker-compose-plugin のパッケージ: 1:docker-ce-cli-24.0.5-1.el7.x86_64
        --> トランザクションの確認を実行しています。
        ---> パッケージ container-selinux.noarch 2:2.119.2-1.911c772.el7_8 を インストール
        ---> パッケージ docker-buildx-plugin.x86_64 0:0.11.2-1.el7 を インストール
        ---> パッケージ docker-ce-rootless-extras.x86_64 0:24.0.5-1.el7 を インストール
        --> 依存性の処理をしています: fuse-overlayfs >= 0.7 のパッケージ: docker-ce-rootless-extras-24.0.5-1.el7.x86_64
        --> 依存性の処理をしています: slirp4netns >= 0.4 のパッケージ: docker-ce-rootless-extras-24.0.5-1.el7.x86_64
        ---> パッケージ docker-compose-plugin.x86_64 0:2.20.2-1.el7 を インストール
        --> トランザクションの確認を実行しています。
        ---> パッケージ fuse-overlayfs.x86_64 0:0.7.2-6.el7_8 を インストール
        --> 依存性の処理をしています: libfuse3.so.3(FUSE_3.2)(64bit) のパッケージ: fuse-overlayfs-0.7.2-6.el7_8.x86_64
        --> 依存性の処理をしています: libfuse3.so.3(FUSE_3.0)(64bit) のパッケージ: fuse-overlayfs-0.7.2-6.el7_8.x86_64
        --> 依存性の処理をしています: libfuse3.so.3()(64bit) のパッケージ: fuse-overlayfs-0.7.2-6.el7_8.x86_64
        ---> パッケージ slirp4netns.x86_64 0:0.4.3-4.el7_8 を インストール
        --> トランザクションの確認を実行しています。
        ---> パッケージ fuse3-libs.x86_64 0:3.6.1-4.el7 を インストール
        --> 依存性解決を終了しました。

        依存性を解決しました

        ================================================================================================
        Package                      アーキテクチャー
                                                バージョン                    リポジトリー         容量
        ================================================================================================
        インストール中:
        containerd.io                x86_64    1.6.22-3.1.el7                docker-ce-stable     34 M
        docker-ce                    x86_64    3:24.0.5-1.el7                docker-ce-stable     24 M
        docker-ce-cli                x86_64    1:24.0.5-1.el7                docker-ce-stable     13 M
        依存性関連でのインストールをします:
        container-selinux            noarch    2:2.119.2-1.911c772.el7_8     extras               40 k
        docker-buildx-plugin         x86_64    0.11.2-1.el7                  docker-ce-stable     13 M
        docker-ce-rootless-extras    x86_64    24.0.5-1.el7                  docker-ce-stable    9.1 M
        docker-compose-plugin        x86_64    2.20.2-1.el7                  docker-ce-stable     13 M
        fuse-overlayfs               x86_64    0.7.2-6.el7_8                 extras               54 k
        fuse3-libs                   x86_64    3.6.1-4.el7                   extras               82 k
        slirp4netns                  x86_64    0.4.3-4.el7_8                 extras               81 k

        トランザクションの要約
        ================================================================================================
        インストール  3 パッケージ (+7 個の依存関係のパッケージ)

        合計容量: 107 M
        総ダウンロード容量: 13 M
        インストール容量: 383 M
        Downloading packages:
        No Presto metadata available for docker-ce-stable
        (1/5): container-selinux-2.119.2-1.911c772.el7_8.noarch.rpm              |  40 kB  00:00:00     
        (2/5): fuse-overlayfs-0.7.2-6.el7_8.x86_64.rpm                           |  54 kB  00:00:00     
        (3/5): fuse3-libs-3.6.1-4.el7.x86_64.rpm                                 |  82 kB  00:00:00     
        (4/5): slirp4netns-0.4.3-4.el7_8.x86_64.rpm                              |  81 kB  00:00:00     
        (5/5): docker-compose-plugin-2.20.2-1.el7.x86_64.rpm                     |  13 MB  00:00:00     
        ------------------------------------------------------------------------------------------------
        合計                                                             15 MB/s |  13 MB  00:00:00     
        Running transaction check
        Running transaction test
        Transaction test succeeded
        Running transaction
        インストール中          : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch             1/10 
        インストール中          : containerd.io-1.6.22-3.1.el7.x86_64                            2/10 
        インストール中          : docker-buildx-plugin-0.11.2-1.el7.x86_64                       3/10 
        インストール中          : docker-compose-plugin-2.20.2-1.el7.x86_64                      4/10 
        インストール中          : 1:docker-ce-cli-24.0.5-1.el7.x86_64                            5/10 
        インストール中          : slirp4netns-0.4.3-4.el7_8.x86_64                               6/10 
        インストール中          : fuse3-libs-3.6.1-4.el7.x86_64                                  7/10 
        インストール中          : fuse-overlayfs-0.7.2-6.el7_8.x86_64                            8/10 
        インストール中          : docker-ce-rootless-extras-24.0.5-1.el7.x86_64                  9/10 
        インストール中          : 3:docker-ce-24.0.5-1.el7.x86_64                               10/10 
        検証中                  : 3:docker-ce-24.0.5-1.el7.x86_64                                1/10 
        検証中                  : fuse3-libs-3.6.1-4.el7.x86_64                                  2/10 
        検証中                  : fuse-overlayfs-0.7.2-6.el7_8.x86_64                            3/10 
        検証中                  : slirp4netns-0.4.3-4.el7_8.x86_64                               4/10 
        検証中                  : 2:container-selinux-2.119.2-1.911c772.el7_8.noarch             5/10 
        検証中                  : docker-compose-plugin-2.20.2-1.el7.x86_64                      6/10 
        検証中                  : 1:docker-ce-cli-24.0.5-1.el7.x86_64                            7/10 
        検証中                  : containerd.io-1.6.22-3.1.el7.x86_64                            8/10 
        検証中                  : docker-buildx-plugin-0.11.2-1.el7.x86_64                       9/10 
        検証中                  : docker-ce-rootless-extras-24.0.5-1.el7.x86_64                 10/10 

        インストール:
        containerd.io.x86_64 0:1.6.22-3.1.el7             docker-ce.x86_64 3:24.0.5-1.el7            
        docker-ce-cli.x86_64 1:24.0.5-1.el7              

        依存性関連をインストールしました:
        container-selinux.noarch 2:2.119.2-1.911c772.el7_8                                            
        docker-buildx-plugin.x86_64 0:0.11.2-1.el7                                                    
        docker-ce-rootless-extras.x86_64 0:24.0.5-1.el7                                               
        docker-compose-plugin.x86_64 0:2.20.2-1.el7                                                   
        fuse-overlayfs.x86_64 0:0.7.2-6.el7_8                                                         
        fuse3-libs.x86_64 0:3.6.1-4.el7                                                               
        slirp4netns.x86_64 0:0.4.3-4.el7_8                                                            

        完了しました!
        ```

- インストール完了の確認。

    ```
    # yum list installed | grep -e "docker" -e "container"
    ```

    - 結果

        ```
        container-selinux.noarch                    2:2.119.2-1.911c772.el7_8  @extras  
        containerd.io.x86_64                        1.6.22-3.1.el7             @docker-ce-stable
        docker-buildx-plugin.x86_64                 0.11.2-1.el7               @docker-ce-stable
        docker-ce.x86_64                            3:24.0.5-1.el7             @docker-ce-stable
        docker-ce-cli.x86_64                        1:24.0.5-1.el7             @docker-ce-stable
        docker-ce-rootless-extras.x86_64            24.0.5-1.el7               @docker-ce-stable
        docker-compose-plugin.x86_64                2.20.2-1.el7               @docker-ce-stable
        ```

- Dockerを起動

    ```
    # systemctl enable docker
    ```
    
    - 結果

        ```
        Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service.
        ```

    ```
    # systemctl start docker
    ```

    - 結果 → なし

        ```
        ```

    ```
    # systemctl status docker    
    ```

    - 結果

        ```
        ● docker.service - Docker Application Container Engine
        Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; vendor preset: disabled)
        Active: active (running) since 木 2023-08-31 13:42:23 JST; 5min ago
            Docs: https://docs.docker.com
        Main PID: 26587 (dockerd)
            Tasks: 23
        Memory: 51.6M
        CGroup: /system.slice/docker.service
                ├─26587 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
                └─26798 /usr/bin/docker-proxy -proto tcp -host-ip 127.0.0.1 -host-port 1514 -conta...

        8月 31 13:42:22 control-plane.minikube.internal dockerd[26587]: time="2023-08-31T13:42:22.1...
        8月 31 13:42:22 control-plane.minikube.internal dockerd[26587]: time="2023-08-31T13:42:22.5...
        8月 31 13:42:22 control-plane.minikube.internal dockerd[26587]: time="2023-08-31T13:42:22.6...
        8月 31 13:42:23 control-plane.minikube.internal dockerd[26587]: time="2023-08-31T13:42:22.9...
        8月 31 13:42:23 control-plane.minikube.internal dockerd[26587]: time="2023-08-31T13:42:23.0...
        8月 31 13:42:23 control-plane.minikube.internal dockerd[26587]: time="2023-08-31T13:42:23.3...
        8月 31 13:42:23 control-plane.minikube.internal dockerd[26587]: time="2023-08-31T13:42:23.4...
        8月 31 13:42:23 control-plane.minikube.internal dockerd[26587]: time="2023-08-31T13:42:23.4...
        8月 31 13:42:23 control-plane.minikube.internal systemd[1]: Started Docker Application Con....
        8月 31 13:42:23 control-plane.minikube.internal dockerd[26587]: time="2023-08-31T13:42:23.4...
        Hint: Some lines were ellipsized, use -l to show in full.
        ```

- バージョンを確認

    ```
    # docker version
    ```

    - 結果

        ```
        Client: Docker Engine - Community
        Version:           24.0.5
        API version:       1.43
        Go version:        go1.20.6
        Git commit:        ced0996
        Built:             Fri Jul 21 20:39:02 2023
        OS/Arch:           linux/amd64
        Context:           default

        Server: Docker Engine - Community
        Engine:
        Version:          24.0.5
        API version:      1.43 (minimum version 1.12)
        Go version:       go1.20.6
        Git commit:       a61e2b4
        Built:            Fri Jul 21 20:38:05 2023
        OS/Arch:          linux/amd64
        Experimental:     false
        containerd:
        Version:          1.6.22
        GitCommit:        8165feabfdfe38c65b599c4993d227328c231fca
        runc:
        Version:          1.1.8
        GitCommit:        v1.1.8-0-g82f18fe
        docker-init:
        Version:          0.19.0
        GitCommit:        de40ad0
        ```
    
## kubectl をバージョンアップする

- [公式ドキュメント](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/)を参考にして、kubctlをバージョンアップする。

- kubectl の実行ファイルの保存先を確認

    ```
    # ll /usr/local/bin/
    ```

    - 結果

        ```
        合計 210980
        -rwxr-xr-x. 1 root root 59383631  8月 13 23:21 docker-compose
        -rwxr-xr-x. 1 root root 46182400  8月 13 07:47 helm
        -rwxr-xr-x. 1 root root 46419968  7月 17 12:13 kubectl
        -rwxr-xr-x. 1 root root 64057293  7月 17 12:13 minikube
        ```

- 以下のコマンドを実行し、kubctlをバージョンアップする。

    ```
    cd ~
    ```

    ```
    rm -f  /usr/local/bin/kubectl
    ```

    ```
    # ll /usr/local/bin/kubectl
    ```
    
    - 結果

        ```
        ls: /usr/local/bin/kubectl にアクセスできません: そのようなファイルやディレクトリはありません
        ```

   - [公式ドキュメント](https://kubernetes.io/ja/docs/tasks/tools/install-kubectl/)より、最新のバージョンのダウンロートコマンドは以下


    ```
    curl -LO "https://dl.k8s.io/release/$(curl -LS https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    ```

    - 結果

        ```
          % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
        100   138  100   138    0     0    380      0 --:--:-- --:--:-- --:--:--   381
        100     7  100     7    0     0     12      0 --:--:-- --:--:-- --:--:--    12
        % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                        Dload  Upload   Total   Spent    Left  Speed
        100   138  100   138    0     0    411      0 --:--:-- --:--:-- --:--:--   410
        100 47.5M  100 47.5M    0     0  14.3M      0  0:00:03  0:00:03 --:--:-- 15.2M
        ```

    ```
    # ls ./kubectl
    ```

    - 結果

        ```
        ./kubectl
        ```

    ```
    # chmod +x ./kubectl
    ```

    ```
    # ll ./kubectl
    ```

    - 結果

        ```
        -rwxr-xr-x. 1 root root 49864704  8月 31 13:58 ./kubectl
        ```

    ```
    mv -f ./kubectl /usr/local/bin
    ```


    ```
    # ll /usr/local/bin/kubectl
    ```

    - 結果

        ```
        -rwxr-xr-x. 1 root root 49864704  8月 31 13:58 /usr/local/bin/kubectl
        ```

- バージョンを確認する

    ```
    # kubectl version --client
    ```

    - 結果

        ```
        Client Version: v1.28.1
        Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
        ```

## minikubeをバージョンアップする

- [公式ドキュメント](https://minikube.sigs.k8s.io/docs/start/)を参考にして、minikubeをインストールする。

- minikube の実行ファイルの保存先を確認


    ```
    # ll /usr/local/bin/
    ```

    - 結果

        ```
        合計 214344
        -rwxr-xr-x. 1 root root 59383631  8月 13 23:21 docker-compose
        -rwxr-xr-x. 1 root root 46182400  8月 13 07:47 helm
        -rwxr-xr-x. 1 root root 49864704  8月 31 13:58 kubectl
        -rwxr-xr-x. 1 root root 64057293  7月 17 12:13 minikube
        ```

- 以下のコマンドを実行し、minikubeをバージョンアップする。

    ```
    # cd ~
    ```

    ```
    # rm -f  /usr/local/bin/minikube
    ```

    ```
    # ls /usr/local/bin/minikube
    ```

    - 結果

        ```
        ls: /usr/local/bin/minikube にアクセスできません: そのようなファイルやディレクトリはありません
        ```

- [公式ドキュメント](https://minikube.sigs.k8s.io/docs/start/)をより、minikubeの最新バージョンのダウンロードコマンドは以下。


    ```
    # curl -Lo minikube  https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    ```

    - 結果

        ```
        % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                    Dload  Upload   Total   Spent    Left  Speed
        100 82.4M  100 82.4M    0     0  16.5M      0  0:00:04  0:00:04 --:--:-- 17.1M
        ```


    ```
    # ls ./minikube
    ```

    - 結果

        ```
        ./minikube
        ```

    ```
    # chmod +x ./minikube
    ```

    ```
    # ll ./minikube
    ```

    - 結果

        ```
        -rwxr-xr-x. 1 root root 86435564  8月 31 14:17 ./minikube
        ```


    ```
    # install ./minikube /usr/local/bin
    ```


    ```
    # ll /usr/local/bin/minikube
    ```

    - 結果

        ```
        -rwxr-xr-x. 1 root root 86435564  8月 31 14:22 /usr/local/bin/minikube
        ```

    ```
    # rm -f ./minikube
    ```

    ```
    # ls ./minikube
    ```

    - 結果

        ```
        ls: ./minikube にアクセスできません: そのようなファイルやディレクトリはありません
        ```

- バージョンを確認

    ```
    # minikube version 
    ```

    - 結果

        ```
        minikube version: v1.31.2
        commit: fd7ecd9c4599bef9f04c0986c4a0187f98a4396e
        ```

## minikube を再起動

- minikube を ストップ

    ```
    # minikube stop 
    ```

    - 結果

        ```
        ✋  「minikube」ノードを停止しています...
        🛑  1 台のノードが停止しました。            
        ```

- minikube を 削除


    ```
    # minikube delete 
    ```

    ```
    🔄  kubeadm を使用して Kubernetes v1.20.7 をアンインストールしています...
    🔥  none の「minikube」を削除しています...
    💀  クラスター「minikube」の全てのトレースを削除しました。
    ```

- minikube を起動

    ```
    # minikube start --vm-driver=none
    ```

    - 結果

        ```
        😄  Centos 7.9.2009 (hyperv/amd64) 上の minikube v1.31.2
        ✨  ユーザーの設定に基づいて none ドライバーを使用します

        ❌  GUEST_MISSING_CONNTRACK が原因で終了します: 申し訳ありませんが、Kubernetes 1.27.4 は root アカウントのパス中にインストールされた crictl が必要です
        ```

## crictl のインストール


- [GitHub](https://github.com/kubernetes-sigs/cri-tools)を参考にした

-以下のコマンドを実行

    ```
    # cd ~
    ```

    ```
    # VERSION="v1.28.0"
    ```

    ```
    # echo $VERSION
    ```

    - 結果

        ```
        v1.28.0
        ```

    ```
    # wget https://github.com/kubernetes-sigs/cri-tools/releases/download/$VERSION/crictl-$VERSION-linux-amd64.tar.gz
    ```

    - 結果

        ```
        --2023-08-31 14:51:16--  https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.28.0/crictl-v1.28.0-linux-amd64.tar.gz
        github.com (github.com) をDNSに問いあわせています... 20.27.177.113
        github.com (github.com)|20.27.177.113|:443 に接続しています... 接続しました。
        HTTP による接続要求を送信しました、応答を待っています... 302 Found
        場所: https://objects.githubusercontent.com/github-production-release-asset-2e65be/80172100/c07e7dde-d128-43aa-b1ec-08484b098c32?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20230831%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20230831T055117Z&X-Amz-Expires=300&X-Amz-Signature=f337bb968200f52dd609788b97214b5f549d80cc0a89a7ec6029a37e1f11aab6&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=80172100&response-content-disposition=attachment%3B%20filename%3Dcrictl-v1.28.0-linux-amd64.tar.gz&response-content-type=application%2Foctet-stream [続く]
        --2023-08-31 14:51:17--  https://objects.githubusercontent.com/github-production-release-asset-2e65be/80172100/c07e7dde-d128-43aa-b1ec-08484b098c32?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20230831%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20230831T055117Z&X-Amz-Expires=300&X-Amz-Signature=f337bb968200f52dd609788b97214b5f549d80cc0a89a7ec6029a37e1f11aab6&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=80172100&response-content-disposition=attachment%3B%20filename%3Dcrictl-v1.28.0-linux-amd64.tar.gz&response-content-type=application%2Foctet-stream
        objects.githubusercontent.com (objects.githubusercontent.com) をDNSに問いあわせています... 185.199.110.133, 185.199.108.133, 185.199.109.133, ...
        objects.githubusercontent.com (objects.githubusercontent.com)|185.199.110.133|:443 に接続しています... 接続しました。
        HTTP による接続要求を送信しました、応答を待っています... 200 OK
        長さ: 23868098 (23M) [application/octet-stream]
        `crictl-v1.28.0-linux-amd64.tar.gz' に保存中

        100%[======================================================>] 23,868,098  11.2MB/s 時間 2.0s   

        2023-08-31 14:51:19 (11.2 MB/s) - `crictl-v1.28.0-linux-amd64.tar.gz' へ保存完了 [23868098/23868098]
        ```

    ```
    # tar zxvf crictl-$VERSION-linux-amd64.tar.gz -C /usr/local/bin
    ```

    - 結果

        ```
        crictl
        ```

    ```
    # ll /usr/local/bin/crictl 
    ```

        - 結果

        ```
        -rwxr-xr-x. 1 kazuhiro users 54939628  8月 14 16:10 /usr/local/bin/crictl
        ```
    
    ```
    # rm -f crictl-$VERSION-linux-amd64.tar.g
    ```

- オーナーをrootにする

    ```
    # chown root:root /usr/local/bin/crictl 
    ```

    ```
    # ll /usr/local/bin/crictl
    ```
     
    - 結果

        ```
        -rwxr-xr-x. 1 root root 54939628  8月 14 16:10 /usr/local/bin/crictl
        ```
        
- バージョンを確認する

    ```
    # crictl -v
    ```

    - 結果

        ```
        crictl version v1.28.0
        ```

## minikubeを起動

- 以下のコマンドを実行

    ```
    # minikube start --vm-driver=none
    ```

    - 結果

        ```
        😄  Centos 7.9.2009 (hyperv/amd64) 上の minikube v1.31.2
        ✨  ユーザーの設定に基づいて none ドライバーを使用します

        🧯  要求された 2200MiB のメモリー割当は、システムのオーバーヘッド (合計システムメモリー: 2909MiB) に十分な空きを残しません。安定性の問題に直面するかも知れません。
        💡  提案: Start minikube with less memory allocated: 'minikube start --memory=2200mb'

        👍  minikube クラスター中のコントロールプレーンの minikube ノードを起動しています
        🤹  localhost (CPU=2、メモリー=2909MB、ディスク=48522MB) 上で実行しています...

        🐳  NOT_FOUND_CRI_DOCKERD が原因で終了します: 

        💡  提案: 

            Kubernetes v1.24+ の none ドライバーと docker container-runtime は cri-dockerd を要求します。
            
            これらの手順を参照して cri-dockerd をインストールしてください:
            
            https://github.com/Mirantis/cri-dockerd
        ```

## cri-dockerd をインストール（準備）

- 以下のサイトに従ってcri-dockerd をインストール

    https://github.com/Mirantis/cri-dockerd


-- 以下のコマンドを実行

    ```
    # cd ~
    ```

    ```
    # git clone https://github.com/Mirantis/cri-dockerd.git
    ```

    - 結果

        ```
        Cloning into 'cri-dockerd'...
        remote: Enumerating objects: 17208, done.
        remote: Counting objects: 100% (17208/17208), done.
        remote: Compressing objects: 100% (7040/7040), done.
        remote: Total 17208 (delta 8710), reused 16872 (delta 8637), pack-reused 0
        Receiving objects: 100% (17208/17208), 40.68 MiB | 6.83 MiB/s, done.
        Resolving deltas: 100% (8710/8710), done.
        ```

### ビルド環境の構築(Go言語のインストール)


- 以下にサイトの手順に従って、ビルド環境を構築

    https://go.dev/doc/install


- 以下のサイトに従って go1.21.0.linux-amd64.tar.gz をダウンロードする

    https://go.dev/doc/install

    https://go.dev/dl/go1.21.0.linux-amd64.tar.gz


- 以下のコマンドを実行

    ```
    # cd ~
    ```

    ```
    # wget https://go.dev/dl/go1.21.0.linux-amd64.tar.gz
    ```

    - 結果

        ```
        --2023-08-31 15:30:22--  https://go.dev/dl/go1.21.0.linux-amd64.tar.gz
        go.dev (go.dev) をDNSに問いあわせています... 216.239.36.21, 216.239.38.21, 216.239.32.21, ...
        go.dev (go.dev)|216.239.36.21|:443 に接続しています... 接続しました。
        HTTP による接続要求を送信しました、応答を待っています... 302 Found
        場所: https://dl.google.com/go/go1.21.0.linux-amd64.tar.gz [続く]
        --2023-08-31 15:30:22--  https://dl.google.com/go/go1.21.0.linux-amd64.tar.gz
        dl.google.com (dl.google.com) をDNSに問いあわせています... 142.250.199.110, 2404:6800:4004:801::200e
        dl.google.com (dl.google.com)|142.250.199.110|:443 に接続しています... 接続しました。
        HTTP による接続要求を送信しました、応答を待っています... 200 OK
        長さ: 66479500 (63M) [application/x-gzip]
        `go1.21.0.linux-amd64.tar.gz' に保存中

        100%[======================================================>] 66,479,500  27.5MB/s 時間 2.3s   

        2023-08-31 15:30:24 (27.5 MB/s) - `go1.21.0.linux-amd64.tar.gz' へ保存完了 [66479500/66479500]
        ```

    ```
    ls go1.21.0.linux-amd64.tar.gz
    ```

    - 結果

        ```
        go1.21.0.linux-amd64.tar.gz
        ```

- Remove any previous Go installation

    ```
    # rm -rf /usr/local/go && tar -C /usr/local -xzf go1.21.0.linux-amd64.tar.gz
    ```

    ```
    # ll /usr/local/go
    ```

    - 結果

        ```
        合計 56
        -rw-r--r--.  1 root root  1337  8月  5 05:14 CONTRIBUTING.md
        -rw-r--r--.  1 root root  1479  8月  5 05:14 LICENSE
        -rw-r--r--.  1 root root  1303  8月  5 05:14 PATENTS
        -rw-r--r--.  1 root root  1455  8月  5 05:14 README.md
        -rw-r--r--.  1 root root   419  8月  5 05:14 SECURITY.md
        -rw-r--r--.  1 root root    35  8月  5 05:14 VERSION
        drwxr-xr-x.  2 root root  4096  8月 31 15:32 api
        drwxr-xr-x.  2 root root    29  8月 31 15:32 bin
        -rw-r--r--.  1 root root    52  8月  5 05:14 codereview.cfg
        drwxr-xr-x.  2 root root   103  8月 31 15:32 doc
        -rw-r--r--.  1 root root   505  8月  5 05:14 go.env
        drwxr-xr-x.  3 root root    18  8月 31 15:32 lib
        drwxr-xr-x.  8 root root   123  8月 31 15:32 misc
        drwxr-xr-x.  4 root root    33  8月 31 15:32 pkg
        drwxr-xr-x. 52 root root  4096  8月 31 15:33 src
        drwxr-xr-x. 26 root root 12288  8月 31 15:33 test
        ```

- 環境変数 $PATH に /usr/local/go/bin を永続化する

    - /etc/profile　をviで開く

        ```
        # vi /etc/profile 
        ```


    - 文末に以下の行を追加し、保存する

        ```
        export PATH=$PATH:/usr/local/go/bin
        ```

    - サーバを再起動する

        ```
        # shutdown -r now
        ```

    - 環境変数 PATH への /usr/local/go/bin の追加を確認する

        ```
        # echo $PATH | grep -e "/usr/local/go/bin"
        ```

        - 結果

            ```
            /root/.krew/bin:/root/.vscode-server/bin/6c3e3dba23e8fadc360aed75ce363ba185c49794/bin/remote-cli:/root/.krew/bin:/root/.krew/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/local/go/bin:/root/bin
            ```

    - Verify that you've installed Go by opening a command prompt and typing the following command

        ```
        # go version
        ```

        - 結果

            ```
            go version go1.21.0 linux/amd64
            ```

### cri-dockerd のインストール

- cri-dockerd を ビルド

    ```
    # cd ~/cri-dockerd/
    ```

    ```
    # make cri-dockerd
    ```

    - 結果（しばらく時間かかる）

        ```
        GOARCH= go build -trimpath -ldflags "-s -w -buildid=`git log -1 --pretty='%h'` -X github.com/Mirantis/cri-dockerd/cmd/version.Version=0.3.4 -X github.com/Mirantis/cri-dockerd/cmd/version.PreRelease=`grep -q dev <<< "0.3.4" && echo "pre" || echo ""` -X github.com/Mirantis/cri-dockerd/cmd/version.GitCommit=`git log -1 --pretty='%h'`" -o cri-dockerd
        ```

- cri-dockerd を インストール

    ```
    # install -o root -g root -m 0755 cri-dockerd /usr/local/bin/cri-dockerd
    ```

    ```
    # install packaging/systemd/* /etc/systemd/system
    ```

    ```
    # sed -i -e 's,/usr/bin/cri-dockerd,/usr/local/bin/cri-dockerd,' /etc/systemd/system/cri-docker.service
    ```

    ```
    # systemctl daemon-reload
    ```

    ```
    # systemctl enable --now cri-docker.socket
    ```

    ```
    # systemctl status cri-docker.socket
    ```

    - 結果

        ```
        ● cri-docker.socket - CRI Docker Socket for the API
        Loaded: loaded (/etc/systemd/system/cri-docker.socket; enabled; vendor preset: disabled)
        Active: active (listening) since 木 2023-08-31 15:56:53 JST; 51s ago
        Listen: /run/cri-dockerd.sock (Stream)

        8月 31 15:56:53 control-plane.minikube.internal systemd[1]: Starting CRI Docker Socket for....
        8月 31 15:56:53 control-plane.minikube.internal systemd[1]: Listening on CRI Docker Socket....
        Hint: Some lines were ellipsized, use -l to show in full.
        ```

## minikubeを起動

- 以下のコマンドを実行

    ```
    # minikube start --vm-driver=none
    ```

    - 結果

        ```
        😄  Centos 7.9.2009 (hyperv/amd64) 上の minikube v1.31.2
        ✨  既存のプロファイルを元に、none ドライバーを使用します

        🧯  要求された 2200MiB のメモリー割当は、システムのオーバーヘッド (合計システムメモリー: 2909MiB) に十分な空きを残しません。安定性の問題に直面するかも知れません。
        💡  提案: Start minikube with less memory allocated: 'minikube start --memory=2200mb'

        👍  minikube クラスター中のコントロールプレーンの minikube ノードを起動しています
        🔄  「minikube」のために既存の none bare metal machine を再起動しています...

        🔗  NOT_FOUND_CNI_PLUGINS が原因で終了します: 


        💡  提案: 

            The none driver with Kubernetes v1.24+ requires containernetworking-plugins.
            
            Please install containernetworking-plugins using these instructions:
            
            https://minikube.sigs.k8s.io/docs/faq/#how-do-i-install-containernetworking-plugins-for-none-driver
        ```

## containernetworking-plugins のインストール

- 以下のサイトに従って実施、

    https://minikube.sigs.k8s.io/docs/faq/#how-do-i-install-containernetworking-plugins-for-none-driver


- 以下のサイトより
    
    https://github.com/containernetworking/plugins/releases

- 以下のコマンドを実行

    ```
    # CNI_PLUGIN_VERSION="v1.3.0"
    ```

    ```
    # CNI_PLUGIN_TAR="cni-plugins-linux-amd64-$CNI_PLUGIN_VERSION.tgz"
    ```

    ```
    # CNI_PLUGIN_INSTALL_DIR="/opt/cni/bin"
    ```

    ```
    # curl -LO "https://github.com/containernetworking/plugins/releases/download/$CNI_PLUGIN_VERSION/$CNI_PLUGIN_TAR"
    ```

    - 結果

        ```
          % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                        Dload  Upload   Total   Spent    Left  Speed
        0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
        100 43.2M  100 43.2M    0     0  11.0M      0  0:00:03  0:00:03 --:--:-- 17.0M
        ```

    ```
    # ll $CNI_PLUGIN_TAR
    ```

    - 結果

        ```
        -rw-r--r--. 1 root root 45338194  8月 31 16:14 cni-plugins-linux-amd64-v1.3.0.tgz
        ```

    ```
    # mkdir -p "$CNI_PLUGIN_INSTALL_DIR"
    ```

    ```
    # ll $CNI_PLUGIN_INSTALL_DIR
    ```

    - 結果

        ```
        合計 0
        ```

    ```
    # tar -xf "$CNI_PLUGIN_TAR" -C "$CNI_PLUGIN_INSTALL_DIR"
    ```

    ```
    # ll $CNI_PLUGIN_INSTALL_DIR 
    ```

    - 結果

        ```
        合計 76636
        -rwxr-xr-x. 1 root root  4016001  5月 10 04:53 bandwidth
        -rwxr-xr-x. 1 root root  4531309  5月 10 04:53 bridge
        -rwxr-xr-x. 1 root root 10816051  5月 10 04:53 dhcp
        -rwxr-xr-x. 1 root root  4171248  5月 10 04:53 dummy
        -rwxr-xr-x. 1 root root  4649749  5月 10 04:53 firewall
        -rwxr-xr-x. 1 root root  4059321  5月 10 04:53 host-device
        -rwxr-xr-x. 1 root root  3444776  5月 10 04:53 host-local
        -rwxr-xr-x. 1 root root  4193323  5月 10 04:53 ipvlan
        -rwxr-xr-x. 1 root root  3514598  5月 10 04:53 loopback
        -rwxr-xr-x. 1 root root  4227193  5月 10 04:53 macvlan
        -rwxr-xr-x. 1 root root  3955775  5月 10 04:53 portmap
        -rwxr-xr-x. 1 root root  4348835  5月 10 04:53 ptp
        -rwxr-xr-x. 1 root root  3716095  5月 10 04:53 sbr
        -rwxr-xr-x. 1 root root  2984504  5月 10 04:53 static
        -rwxr-xr-x. 1 root root  4258344  5月 10 04:53 tap
        -rwxr-xr-x. 1 root root  3603365  5月 10 04:53 tuning
        -rwxr-xr-x. 1 root root  4187498  5月 10 04:53 vlan
        -rwxr-xr-x. 1 root root  3754911  5月 10 04:53 vrf
        ```

    ```
    # rm "$CNI_PLUGIN_TAR"
    ```

## minikubeを起動

- minikubを起動

    ```
    # minikube start --vm-driver=none
    ```

    - 結果

        ```
        😄  Centos 7.9.2009 (hyperv/amd64) 上の minikube v1.31.2
        ✨  既存のプロファイルを元に、none ドライバーを使用します

        🧯  要求された 2200MiB のメモリー割当は、システムのオーバーヘッド (合計システムメモリー: 2909MiB) に十分な空きを残しません。安定性の問題に直面するかも知れません。
        💡  提案: Start minikube with less memory allocated: 'minikube start --memory=2200mb'

        👍  minikube クラスター中のコントロールプレーンの minikube ノードを起動しています
        🔄  「minikube」のために既存の none bare metal machine を再起動しています...
        ℹ️  OS リリースは CentOS Linux 7 (Core) です
        E0831 16:32:20.919444   30890 start.go:415] unable to disable preinstalled bridge CNI(s): failed to disable all bridge cni configs in "/etc/cni/net.d": sudo find /etc/cni/net.d -maxdepth 1 -type f ( ( -name *bridge* -or -name *podman* ) -and -not -name *.mk_disabled ) -printf "%p, " -exec sh -c "sudo mv {} {}.mk_disabled" ;: exit status 1
        stdout:

        stderr:
        find: ‘/etc/cni/net.d’: そのようなファイルやディレクトリはありません
            > kubeadm.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
            > kubectl.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
            > kubelet.sha256:  64 B / 64 B [-------------------------] 100.00% ? p/s 0s
            > kubectl:  46.98 MiB / 46.98 MiB [-------------] 100.00% 7.29 MiB p/s 6.6s
            > kubeadm:  45.93 MiB / 45.93 MiB [-------------] 100.00% 7.04 MiB p/s 6.7s
            > kubelet:  101.25 MiB / 101.25 MiB [----------] 100.00% 10.78 MiB p/s 9.6s

            ▪ 証明書と鍵を作成しています...
            ▪ コントロールプレーンを起動しています...
            ▪ RBAC のルールを設定中です...
        🔗  bridge CNI (コンテナーネットワークインターフェース) を設定中です...
        🤹  ローカルホスト環境を設定中です...

        ❗  'none' ドライバーは既存 VM の統合が必要なエキスパートに向けて設計されています。
        💡  多くのユーザーはより新しい 'docker' ドライバーを代わりに使用すべきです (root 権限が必要ありません！)
        📘  追加の詳細情報はこちらを参照してください: https://minikube.sigs.k8s.io/docs/reference/drivers/none/

        ❗  kubectl と minikube の構成は /root に保存されます
        ❗  kubectl か minikube コマンドを独自のユーザーとして使用するためには、そのコマンドの再配置が必要な場合があります。たとえば、独自の設定を上書きするためには、以下を実行します

            ▪ sudo mv /root/.kube /root/.minikube $HOME
            ▪ sudo chown -R $USER $HOME/.kube $HOME/.minikube

        💡  これは環境変数 CHANGE_MINIKUBE_NONE_USER=true を設定して自動的に行うこともできます
        🔎  Kubernetes コンポーネントを検証しています...
            ▪ gcr.io/k8s-minikube/storage-provisioner:v5 イメージを使用しています
        🌟  有効なアドオン: default-storageclass, storage-provisioner
        🏄  終了しました！kubectl がデフォルトで「minikube」クラスターと「default」ネームスペースを使用するよう設定されました
        ```


    ```
    # minikube status 
    ```

    - 結果

        ```
        minikube
        type: Control Plane
        host: Running
        kubelet: Running
        apiserver: Running
        kubeconfig: Configured
        ```
    
- minikube を削除

    ```
    # minikube delete 
    ```

    - 結果

        ```
        🔄  kubeadm を使用して Kubernetes v1.27.4 をアンインストールしています...
        🔥  none の「minikube」を削除しています...
        💀  クラスター「minikube」の全てのトレースを削除しました。
        ```

- minikubを起動

    ```
    # minikube start --vm-driver=none
    ```

    - 結果

        ```
        😄  Centos 7.9.2009 (hyperv/amd64) 上の minikube v1.31.2
        ✨  ユーザーの設定に基づいて none ドライバーを使用します

        🧯  要求された 2200MiB のメモリー割当は、システムのオーバーヘッド (合計システムメモリー: 2909MiB) に十分な空きを残しません。安定性の問題に直面するかも知れません。
        💡  提案: Start minikube with less memory allocated: 'minikube start --memory=2200mb'

        👍  minikube クラスター中のコントロールプレーンの minikube ノードを起動しています
        🤹  localhost (CPU=2、メモリー=2909MB、ディスク=48522MB) 上で実行しています...
        ℹ️  OS リリースは CentOS Linux 7 (Core) です
        🐳  Docker 24.0.5 で Kubernetes v1.27.4 を準備しています...
            ▪ 証明書と鍵を作成しています...
            ▪ コントロールプレーンを起動しています...
            ▪ RBAC のルールを設定中です...
        🔗  bridge CNI (コンテナーネットワークインターフェース) を設定中です...
        🤹  ローカルホスト環境を設定中です...

        ❗  'none' ドライバーは既存 VM の統合が必要なエキスパートに向けて設計されています。
        💡  多くのユーザーはより新しい 'docker' ドライバーを代わりに使用すべきです (root 権限が必要ありません！)
        📘  追加の詳細情報はこちらを参照してください: https://minikube.sigs.k8s.io/docs/reference/drivers/none/

        ❗  kubectl と minikube の構成は /root に保存されます
        ❗  kubectl か minikube コマンドを独自のユーザーとして使用するためには、そのコマンドの再配置が必要な場合があります。たとえば、独自の設定を上書きするためには、以下を実行します

            ▪ sudo mv /root/.kube /root/.minikube $HOME
            ▪ sudo chown -R $USER $HOME/.kube $HOME/.minikube

        💡  これは環境変数 CHANGE_MINIKUBE_NONE_USER=true を設定して自動的に行うこともできます
        🔎  Kubernetes コンポーネントを検証しています...
            ▪ gcr.io/k8s-minikube/storage-provisioner:v5 イメージを使用しています
        🌟  有効なアドオン: default-storageclass, storage-provisioner
        🏄  終了しました！kubectl がデフォルトで「minikube」クラスターと「default」ネームスペースを使用するよう設定されました
        ```

- とりあえずこれで使ってみる。

## docker-compose のバージョン

docker-compose は 2023-08-15現在の最新のため、このままとする


## Concourse CI のインストール

-  Concourse CI のインストール手順に従い、インストールする

    https://note.com/shift_tech/n/nead03c02b095


- 以下を実行

    ```
    # cd ~
    ```

    ```
    # git clone git@github.com:concourse/concourse-docker.git
    ```

- docker-compose.yml をコピー

    - [参考にしたサイト](https://note.com/shift_tech/n/nead03c02b095)では、Concourse CIのコンテナが正常動作しなかった（http通信ができない模様）。

    - このため、docker-compose.ymlは、[GitHub](https://github.com/concourse/concourse-docker)から取得する（これにより、Concourse CIにてhttp通信ができた）

    ```
    # cd ~/concourse-install
    ```

    ```
    # cp ~/concourse-docker/docker-compose.yml .
    ```

    ```
    # cp -r ~/concourse-docker/keys/ .
    ```

    ```
    # ll .
    ```

    - 結果

        ```
        rw-r--r--. 1 root root 94098  8月 31 16:52 README.md
        -rw-r--r--. 1 root root  1323  8月 31 20:22 docker-compose.yml
        drwxr-xr-x. 4 root root    47  8月 31 20:24 keys
        ```

    ```
    # ll ./keys
    ```

        - 結果

        ```
        合計 4
        -rwxr-xr-x. 1 root root 617  8月 31 20:24 generate
        drwxr-xr-x. 2 root root  22  8月 31 20:24 web
        drwxr-xr-x. 2 root root  22  8月 31 20:24 worker
        ```
- generate を実行

    ```
    # ./keys/generate 
    ```

    - 結果

        ```
        Unable to find image 'concourse/concourse:latest' locally
        latest: Pulling from concourse/concourse
        ba949eef9048: Pull complete 
        44249e2c30e6: Pull complete 
        e1e412eca9c2: Pull complete 
        a0041624f8fc: Pull complete 
        909250c2dfce: Pull complete 
        eb0bf7bd89ba: Pull complete 
        ebf39e61dec7: Pull complete 
        ec17eb4c09e6: Pull complete 
        cb7b79e6ec50: Pull complete 
        Digest: sha256:e77630abc5021c5a8e22b86ac107843a998164d129d4bbd47735ec9b7743eff5
        Status: Downloaded newer image for concourse/concourse:latest
        wrote private key to /keys/session_signing_key
        wrote private key to /keys/tsa_host_key
        wrote ssh public key to /keys/tsa_host_key.pub
        wrote private key to /keys/worker_key
        wrote ssh public key to /keys/worker_key.pub
        ```

- ここまででConcourse CIのインストールの準備は完了です。

##  Concourse CI を起動

- 以下のコマンドを実行

    ```
    # cd ~/concourse-install
    ```

    ```
    # docker-compose up -d
    ```

    - 結果

        ```
        [+] Running 14/14
        ✔ db 13 layers [⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                    21.4s 
        ✔ 52d2b7f179e3 Pull complete                                                            2.3s 
        ✔ d9c06b35c8a5 Pull complete                                                            0.7s 
        ✔ ec0d4c36c7f4 Pull complete                                                            0.9s 
        ✔ aa8e32a16a69 Pull complete                                                            2.5s 
        ✔ 8950a67e90d4 Pull complete                                                            2.2s 
        ✔ 1b47429b7c5f Pull complete                                                            2.9s 
        ✔ a773f7da97bb Pull complete                                                            3.0s 
        ✔ 7bddc9bbcf13 Pull complete                                                            3.1s 
        ✔ 60829730fa39 Pull complete                                                            7.1s 
        ✔ f3d9c845d2f3 Pull complete                                                            8.6s 
        ✔ cfcd43fe346d Pull complete                                                            8.6s 
        ✔ 576335d55cdb Pull complete                                                            8.6s 
        ✔ caad4144446c Pull complete                                                            9.3s 
        [+] Running 4/4
        ✔ Network concourse-install_default     Created                                           0.2s 
        ✔ Container concourse-install-db-1      Started                                           0.4s 
        ✔ Container concourse-install-web-1     Started                                           0.1s 
        ✔ Container concourse-install-worker-1  Started                                           0.1s         
        ```

- curl で `http://localhost:8080`にアクセスする

    ```
    # curl http://localhost:8080
    ```

    - なぜか、curl でアクセスすることで、ホストOSのブラウザでアクセスできるようになった

- ホストOSのブラウザで`http://localhost:8080`にアクセスする

-  Concourse CI のインストール手順(簡単な操作)

    https://note.com/shift_tech/n/nead03c02b095


- Concourse CI のチュートリアル

    https://concoursetutorial-ja.site.lkj.io/


##  flyコマンドのインストール

- 以下のサイトを参考にしてインストール

    https://github.com/concourse/concourse/tree/master/fly

- Fly は Concourse UI の右下隅からダウンロードできます

- リンクは以下

    http://localhost:8080/api/v1/cli?arch=amd64&platform=linux


- ここでは、ダウンロードした「fly」ファイルを`~`に置いた

- fly をインストールする

    ```
    # cd ~
    ```

    ```
    # install fly /usr/local/bin
    ```

- パスが通っているかを確認

    ```
    # which fly
    ```

    - 結果

        ```
        /usr/local/bin/fly
        ```

- バージョンを確認

    ```
    # fly -v
    ```
    
    - 結果

        ```
        7.10.0
        ```

## 申送り事項

- 以下の件が未解決と認識


1. helm はまだ使用していないため、削除したい

    ```
    # ll /usr/local/bin/
    合計 404224
    -rwxr-xr-x. 1 root root 41664512  8月 31 15:55 cri-dockerd
    -rwxr-xr-x. 1 root root 54939628  8月 14 16:10 crictl
    -rwxr-xr-x. 1 root root 59383631  8月 13 23:21 docker-compose
    -rwxr-xr-x. 1 root root 75450698  8月 31 21:00 fly
    -rwxr-xr-x. 1 root root 46182400  8月 13 07:47 helm
    -rwxr-xr-x. 1 root root 49864704  8月 31 13:58 kubectl
    -rwxr-xr-x. 1 root root 86435564  8月 31 14:22 minikube
    ```

1. minikube の起動時に表示される以下の推奨メッセージに対しての対応

    ```
    ❗  'none' ドライバーは既存 VM の統合が必要なエキスパートに向けて設計されています。
    💡  多くのユーザーはより新しい 'docker' ドライバーを代わりに使用すべきです (root 権限が必要ありません！)
    📘  追加の詳細情報はこちらを参照してください: https://minikube.sigs.k8s.io/docs/reference/drivers/none/

    ```

1. minikube の起動時に表示される以下の推奨メッセージに対しての対応

    ```
    ❗  kubectl と minikube の構成は /root に保存されます
    ❗  kubectl か minikube コマンドを独自のユーザーとして使用するためには、そのコマンドの再配置が必要な場合があります。たとえば、独自の設定を上書きするためには、以下を実行します

        ▪ sudo mv /root/.kube /root/.minikube $HOME
        ▪ sudo chown -R $USER $HOME/.kube $HOME/.minikube

    💡  これは環境変数 CHANGE_MINIKUBE_NONE_USER=true を設定して自動的に行うこともできます
    ```

## Concourse CI の操作

- 「Concourse CI のチュートリアル」を参考にした

    https://concoursetutorial-ja.site.lkj.io/


### concourse のログインにいて

- 以下とサイトの情報を元にとりあえず、暫定のログイン手順を考えた

    https://github.com/concourse/concourse-docker

    https://note.com/shift_tech/n/nead03c02b095

    https://concoursetutorial-ja.site.lkj.io/


- Concourse CI のデフォルトユーザは、ユーザID: `test`、パスワード: `test`

- `target=tutorial` として以下のコマンドを実行する

    ```
    # fly --target=tutorial login --concourse-url=http://localhost:8080 --username=test --password=test
    ```

    - 結果

        ```
        logging in to team 'main'


        target saved
        ```

- 以下の内容で、`~/.flyc` が作成される。

    ```
    # cat ~/.flyrc 
    ```
    
    - 結果

        ```
        targets:
        tutorial:
            api: http://localhost:8080
            team: main
            token:
            type: bearer
            value: IsRCaqdj0owOQZAvpL6mkb4OMWGQ1PFkAAAAAA
        ```

- ここでとりあえず以下のコマンドを実行してみる(理由は不明)

    ```
    # fly --target=tutorial sync
    ```

    - 結果

        ```
        version 7.10.0 already matches; skipping
        ```

- 更に以下のコマンドを実行する（Concourse CI のWebUIへのログイン）(★２回目以降のログインはここから実行)

    ```
    # fly --target tutorial login --concourse-url http://localhost:8080
    ```

    - 操作
    - `http://localhost:8080/login?fly_port=43269` でホストOSのブラウザにアクセスし、表示されたtokenを貼り付ける
    - ユーザID: `test`、パスワード: `test` とする
    - 上記操作をすると、Concourse CI のWeb UIにログインできる

        ```
        logging in to team 'main'

        navigate to the following URL in your browser:

        http://localhost:8080/login?fly_port=43269

        or enter token manually (input hidden): 
        target saved
        ```

## Concourse CI のチュートリアル実施

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/task-hello-world


### Hello World

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/task-hello-world


- 以下のコマンドを実行

    ```
    # cd ~
    ```

    ```
    git clone https://github.com/starkandwayne/concourse-tutorial.git
    ```
    
    ```
    cd concourse-tutorial/tutorials/basic/task-hello-world
    ```

    ```
    fly -t tutorial execute -c task_hello_world.yml
    ```

###  Task inputs について

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/task-inputs


### Task スクリプトを別ファイルとして指定する

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/task-scripts


### ベーシックなパイプライン

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/basic-pipeline


- パイプラインを作成する

    ```
    # cd ../basic-pipeline
    ```

    ```
    # fly -t tutorial set-pipeline -c pipeline.yml -p hello-world
    ```

- パイラインを作成する


    ```
    # fly -t tutorial unpause-pipeline -p hello-world
    ```

    ```
    # fly -t tutorial unpause-job --job hello-world/job-hello-world
    ```

### Resource について

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/pipeline-resources


### Job の出力結果をターミナルで確認する

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/watch-job-output


### Job を起動する

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/trigger-jobs


- 以下のコマンドで、Pipeline hello-world の job-hello-world をもう1度起動することができます:

    ```
    # fly -t tutorial trigger-job -j hello-world/job-hello-world
    ```

- Job が実行されている間（そして完了した後）、ターミナルで fly watch コマンドを使ってログの出力結果を見ることができます:

    ```
    # fly -t tutorial watch -j hello-world/job-hello-world
    ```

- あるいは、2つのコマンドを組み合わせることもできます - Jobを起動し、trigger-job -w フラグで出力を監視します:

    ```
    # fly -t tutorial trigger-job -j hello-world/job-hello-world -w
    ```

### Resource を使って Job を起動する

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/triggers

### パイプラインを削除する

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/destroy-pipelines


- 以下のコマンドを実行する

    ```
    # fly -t tutorial destroy-pipeline -p hello-world
    ```

### Task で Resource から読み込んだファイルを利用する

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/job-inputs



### 成功した Task の outputs を別の Task の inputs にする

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/task-outputs-to-inputs


### ビルドの成果物をアップロードする

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/publishing-outputs

- 「ビルドの成果物をアップロードす」は`pipeline.yml`を以下のように修正したないと、成功しなかった

- 以下のコマンドを実行

    ```
    # cd ~/concourse-tutorial/tutorials/basic/publishing-outputs/
    ```
    
    ~~cp pipeline-missing-credentials.yml pipeline.yml~~

    ```
    # fly -t tutorial set-pipeline -p publishing-outputs -c pipeline.yml
    ```
    
    ```
    # fly -t tutorial unpause-pipeline -p publishing-outputs
    ```

- オレンジにならないが、ハンズオンをこのまま実施

    ![オレンジにならない](./imgs/オレンジにならない.png)


~~    # fly -t tutorial unpause-job -j publishing-outputs/job-bump-date~~

~~- パイプラインを削除する~~

~~    # fly -t tutorial destroy-pipeline -p publishing-outputs~~

- 発生したエラー

    ```
    # fly -t tutorial check-resource -r publishing-outputs/resource-gist
    checking publishing-outputs/resource-gist in build 1209
    initializing check: resource-gist
    selected worker: 971dfbb64a5a
    Error loading key "/tmp/git-resource-private-key": error in libcrypto
    failed
    ```

    - 対策 → `private_key: |`の上に空行を入れた（理由は不明）
    - 【後で気付いたが】この対策は行わなくてもOKな模様、秘密鍵に関する問題がなぜ発生するかは不明

- 発生したエラー

    ```
    # fly -t tutorial check-resource -r publishing-outputs/resource-gist
    checking publishing-outputs/resource-gist in build 1274
    initializing check: resource-gist
    selected worker: 971dfbb64a5a
    Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
    Cloning into '/tmp/git-resource-repo-cache'...
    warning: Could not find remote branch master to clone.
    fatal: Remote branch master not found in upstream origin
    failed    
    ```

    - 対策 → `branch: master` をコメントアウトした（`gist`` にはブランチが存在しないため、そのようにした）

- 発生したエラー

    ```
    # fly -t tutorial trigger-job -j publishing-outputs/job-bump-date -w
    started publishing-outputs/job-bump-date #2

    selected worker: 971dfbb64a5a
    INFO: found existing resource cache

    selected worker: 971dfbb64a5a
    Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
    Cloning into '/tmp/build/get'...
    19482e7 
    initializing
    initializing check: image
    selected worker: 971dfbb64a5a
    failed to fetch digest for image 'starkandwayne/concourse:latest': 401 Unauthorized
    does the image exist?
    image check failed
    errored
    ```

    - 対策 → コンテナイメージを`bitnami/concourse`に変更した。（`starkandwayne/concourse`がDockerHubに存在しないため）


- 以下のコマンドを実行

- パイプラインを削除する

    ```
    # fly -t tutorial destroy-pipeline -p publishing-outputs
    ```

- パイプラインを作成

    ```
    # fly -t tutorial set-pipeline -p publishing-outputs -c pipeline.yml
    ```

- パイプラインのリソースをチェック

    ```
    # fly -t tutorial check-resource -r publishing-outputs/resource-gist
    ```

- パイプラインを有効化
- ※この手順は、オリジナルにはない

    ```
    # fly -t tutorial unpause-pipeline -p publishing-outputs
    ```

- パイプラインを実行し、動作を監視

    ```
    # fly -t tutorial trigger-job -j publishing-outputs/job-bump-date -w
    ```


- 発生したエラー
 
    ```
    # fly -t tutorial trigger-job -j publishing-outputs/job-bump-date -w
    started publishing-outputs/job-bump-date #1

    selected worker: 971dfbb64a5a
    INFO: found existing resource cache

    selected worker: 971dfbb64a5a
    Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
    Cloning into '/tmp/build/get'...
    19482e7 
    initializing
    initializing check: image
    selected worker: 971dfbb64a5a
    selected worker: 971dfbb64a5a
    waiting for docker to come up...
    Pulling bitnami/concourse@sha256:d014b25bed090db1b4410d233047916f09c78e7c7e2d24e46ae2f140578e3ef4...
    docker.io/bitnami/concourse@sha256:d014b25bed090db1b4410d233047916f09c78e7c7e2d24e46ae2f140578e3ef4: Pulling from bitnami/concourse
    9d4a76cdb52b: Pulling fs layer
    9d4a76cdb52b: Verifying Checksum
    9d4a76cdb52b: Download complete
    9d4a76cdb52b: Pull complete
    Digest: sha256:d014b25bed090db1b4410d233047916f09c78e7c7e2d24e46ae2f140578e3ef4
    Status: Downloaded newer image for bitnami/concourse@sha256:d014b25bed090db1b4410d233047916f09c78e7c7e2d24e46ae2f140578e3ef4
    docker.io/bitnami/concourse@sha256:d014b25bed090db1b4410d233047916f09c78e7c7e2d24e46ae2f140578e3ef4

    Successfully pulled bitnami/concourse@sha256:d014b25bed090db1b4410d233047916f09c78e7c7e2d24e46ae2f140578e3ef4.

    selected worker: 971dfbb64a5a
    running resource-tutorial/tutorials/basic/publishing-outputs/bump-timestamp-file.sh
    + git clone resource-gist updated-gist
    resource-tutorial/tutorials/basic/publishing-outputs/bump-timestamp-file.sh: 6: git: not found
    failed
    ```
    - 原因（コンテナイメージを`bitnami/concourse`に`git`がイントールされていない？）

    - 対策 → `git`　がインストールされているイメージを使用すればい？
    - 以下のサイト
        https://blog.ik.am/entries/380
    - より、コンテナイメージ `getourneau/alpine-bash-git`は、`bash`と`git`をインストール済の模様

    - よって、使用するコンテナを`getourneau/alpine-bash-git`に変える


- 以下のコマンドを実行

- パイプラインを削除する

    ```
    # fly -t tutorial destroy-pipeline -p publishing-outputs
    ```

- パイプラインを作成

    ```
    # fly -t tutorial set-pipeline -p publishing-outputs -c pipeline.yml
    ```

- パイプラインのリソースをチェック

    ```
    # fly -t tutorial check-resource -r publishing-outputs/resource-gist
    ```

- パイプラインを有効化
- ※この手順は、オリジナルにはない

    ```
    # fly -t tutorial unpause-pipeline -p publishing-outputs
    ```

- パイプラインを実行し、動作を監視

    ```
    # fly -t tutorial trigger-job -j publishing-outputs/job-bump-date -w
    ```

- 発生したエラー

    ```
    # fly -t tutorial trigger-job -j publishing-outputs/job-bump-date -w
    started publishing-outputs/job-bump-date #1

    selected worker: 971dfbb64a5a
    INFO: found existing resource cache

    selected worker: 971dfbb64a5a
    Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
    Cloning into '/tmp/build/get'...
    19482e7 
    initializing
    initializing check: image
    selected worker: 971dfbb64a5a
    selected worker: 971dfbb64a5a
    waiting for docker to come up...
    Pulling getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7...
    docker.io/getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7: Pulling from getourneau/alpine-bash-git
    4fe2ade4980c: Pulling fs layer
    03c196859ec8: Pulling fs layer
    720d2de11875: Pulling fs layer
    4fe2ade4980c: Verifying Checksum
    4fe2ade4980c: Download complete
    03c196859ec8: Verifying Checksum
    03c196859ec8: Download complete
    4fe2ade4980c: Pull complete
    03c196859ec8: Pull complete
    720d2de11875: Verifying Checksum
    720d2de11875: Download complete
    720d2de11875: Pull complete
    Digest: sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7
    Status: Downloaded newer image for getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7
    docker.io/getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7

    Successfully pulled getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7.

    selected worker: 971dfbb64a5a
    running resource-tutorial/tutorials/basic/publishing-outputs/bump-timestamp-file.sh
    + git clone resource-gist updated-gist
    Cloning into 'updated-gist'...
    done.
    + cd updated-gist
    + date
    + echo Fri Sep 1 13:19:28 UTC 2023
    + git config --global user.email nobody@concourse-ci.org
    + git config --global user.name Concourse
    + git add .
    + git commit -m 'Bumped date'
    [main 88cf34b] Bumped date
    1 file changed, 1 insertion(+), 1 deletion(-)
    selected worker: 971dfbb64a5a
    Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
    invalid payload (missing branch)
    failed
    ```

- 対策 `pipeline.yml`を以下のように変更（`master`が無いなら`main`？）

    ```
    source:
    # branch: master
    branch: main
    uri: git@gist.github.com:5e40d8df48fa30baf9abee4b35a2241a.git
    ```

- 以下のコマンドを実行

- パイプラインを削除する

    ```
    # fly -t tutorial destroy-pipeline -p publishing-outputs
    ```

- パイプラインを作成

    ```
    # fly -t tutorial set-pipeline -p publishing-outputs -c pipeline.yml
    ```

- パイプラインのリソースをチェック

    ```
    # fly -t tutorial check-resource -r publishing-outputs/resource-gist
    ```

- パイプラインを有効化
- ※この手順は、オリジナルにはない

    ```
    # fly -t tutorial unpause-pipeline -p publishing-outputs
    ```

- パイプラインを実行し、動作を監視

    ```
    # fly -t tutorial trigger-job -j publishing-outputs/job-bump-date -w
    ```

- 正常動作した！！

    ```
   # fly -t tutorial trigger-job -j publishing-outputs/job-bump-date -w
    started publishing-outputs/job-bump-date #1

    selected worker: 971dfbb64a5a
    INFO: found existing resource cache

    selected worker: 971dfbb64a5a
    Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
    Cloning into '/tmp/build/get'...
    19482e7 
    initializing
    initializing check: image
    selected worker: 971dfbb64a5a
    selected worker: 971dfbb64a5a
    waiting for docker to come up...
    Pulling getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7...
    docker.io/getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7: Pulling from getourneau/alpine-bash-git
    4fe2ade4980c: Pulling fs layer
    03c196859ec8: Pulling fs layer
    720d2de11875: Pulling fs layer
    4fe2ade4980c: Verifying Checksum
    4fe2ade4980c: Download complete
    03c196859ec8: Verifying Checksum
    03c196859ec8: Download complete
    4fe2ade4980c: Pull complete
    03c196859ec8: Pull complete
    720d2de11875: Download complete
    720d2de11875: Pull complete
    Digest: sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7
    Status: Downloaded newer image for getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7
    docker.io/getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7

    Successfully pulled getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7.

    selected worker: 971dfbb64a5a
    running resource-tutorial/tutorials/basic/publishing-outputs/bump-timestamp-file.sh
    + git clone resource-gist updated-gist
    Cloning into 'updated-gist'...
    done.
    + cd updated-gist
    + date
    + echo Fri Sep 1 13:41:00 UTC 2023
    + git config --global user.email nobody@concourse-ci.org
    + git config --global user.name Concourse
    + git add .
    + git commit -m 'Bumped date'
    [main e1ccccf] Bumped date
    1 file changed, 1 insertion(+), 1 deletion(-)
    selected worker: 971dfbb64a5a
    Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
    To gist.github.com:5e40d8df48fa30baf9abee4b35a2241a.git
    19482e7..e1ccccf  HEAD -> main
    selected worker: 971dfbb64a5a
    Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
    Cloning into '/tmp/build/get'...
    e1ccccf Bumped date
    succeeded 
    ```

- 成功した`pipeline.yml`は以下

    ```
    # cat pipeline.yml
    ---
    resources:
    - name: resource-tutorial
        type: git
        source:
        uri: https://github.com/starkandwayne/concourse-tutorial.git
        branch: develop
    - name: resource-gist
        type: git
        source:
        # branch: master
        branch: main
        uri: git@gist.github.com:5e40d8df48fa30baf9abee4b35a2241a.git
        private_key: |
            -----BEGIN RSA PRIVATE KEY-----
            MIIEowIBAAKCAQEAp/Tcj0IhF9PG27xq6dH4G72Nqns0KjaFnBABLEZty8DtFiaS
            oESMZgrsrEJe36l1kaE9i/3UWKYrU5LZsDvn8DVc4Y5U0IE/QcmsplH5G0zSA0VJ
            VRLB4higtJV4JPXKyjS1MBG5pLAD7pUbQ+jBj3a6icsHEuSGAfssY2lpSj8UOUwd
            fmnPaaXmyY3x1K1KuaggQ8mLOJNPAJ4x2jar+bap0URlggbu2kFQ4/XakDZVCRxT
            JDybRM37RwrSqLYspnOsaJ18z2EJCuIaiVcySyz7DjVcPTdXQzjkWw8nYPX5IykN
            zHfHC6qgCcN4AJlgFZl1F9xisXnD+B5X5R4LWwIDAQABAoIBAAQc/g3QG8lemV8m
            RSQGzWG4ibCkJcnm3ezNg4nXC7dSuTuypCKiqyGQoO0zDunBV6zCWySDieDF6Qe5
            7/Td8rcyR10KxE7661asHrtQBJ7Did0kpEAeHntwCPeDNZcKIfZDxjAwLvC2ktIT
            +r/2Ak+GI9leDIVM7W88/IBOw5Ja4POYZUpJ+v6ym3kEA+Xgrw5ej0+pOO2NmTFL
            pujQd0P1MnK9kaftRPBg9OlFpq0wjViLDPZKcb9XzHBx6LYHqE2IG7P93b8hMNqP
            A8HAcH8S1h7k3nWT20x/eTVWFTNtGoAsO31nz6QH+E6JbFs6ApNpiON30HkDSFX5
            wnhbsikCgYEA2gCzC01cJSdzp/yZexelM99ivpwEmytE6R4qBemiNAcRwmKTYSeA
            ct3AnxPBj1vIaTgwtq9P/JBEPQMonhTWq3bj1v+70HdBz50njd0Rl58zC9rx6nF4
            2zNOTga3EcmQ2EyeZjxFVSs8v1YtHgT1OUjkcFnS2paBrKkiLhd38WcCgYEAxTsY
            NkoNTQBitORS5wtp39Dr87k6tyVOmRn8MZuBeaKAJ2cIXFHDwRtNm5so+jQS63td
            QVOu4wF9fxBPHrxT4z/D9ASfrmsc1Z7gTRGu/sHGYmdDOomAI6RKBBjNtfgBIekW
            HNrLt+xLXxr8xL7QC8bJtKwPmigWVzo8DWSrme0CgYACrKeFp/lNa2J72Rl47R1V
            uZPYislzreA2i+wwDmGzCbMqE1ODiZyFzDqkuPVS8OlQgSP32ca9bnen1/YTmmXX
            zKmW5aRENnJUPbVShDfHCGjz6Ee3fJTi+4omYua0DSj9vlLjJjIjjVg9cK01BRKN
            FVvYFQIFNHt6xshokFkkWQKBgQCYVS48MDHZuWSDhp4paX1aqwiy8+vPrPbp9VH+
            FreH9OS6ii/A7j4dljL47nxV04aRbnT2keXP20TMsRILETZRnNyCSlfy5TQeIlnn
            7LKWfZ/2PP+F5NGdtbSdOXMZCvYE9PxpSOxzoAQO7s8wPph9oAoGi6Z5UGEA+i+L
            wKdxeQKBgDqFNCOKd50GzA5tavw06AzzsHzYc1PoJBIw3NphYkJimxPbI8RBkkCa
            HsMd73mr7LrLgzZWj7gBD5GXzBqHDdm+/8u4ev3dhquGeBprFJI8E7K7TEi2Spwz
            rm8Hl2L077bV+VOTb177RetIFLt/dY8KREWC72vvqxVIzalhjDnG
            -----END RSA PRIVATE KEY-----

    jobs:
    - name: job-bump-date
        serial: true
        plan:
        - get: resource-tutorial
        - get: resource-gist
        - task: bump-timestamp-file
            config:
            platform: linux
            image_resource:
                type: docker-image
                # source: {repository: starkandwayne/concourse}
                # source: {repository: bitnami/concourse}
                source: {repository: getourneau/alpine-bash-git}

            inputs:
                - name: resource-tutorial
                - name: resource-gist
            outputs:
                - name: updated-gist
            run:
                path: resource-tutorial/tutorials/basic/publishing-outputs/bump-timestamp-file.sh

        - put: resource-gist
            params:
            repository: updated-gist
    ```

### 申送り事項

- `starkandwayne/concourse` はDockerHUBに無いが、以下のサイトで作成する模様
　　
    https://github.com/Qarik-Group/dockerfiles/tree/master/concourse-kubernetes/latest

    https://github.com/Qarik-Group/dockerfiles

    https://concoursetutorial-ja.site.lkj.io/basics/publishing-outputs

- 今後、Docker Images `starkandwayne/concourse`  を作成し、再チャレンジしてみる

### パラメータを利用する

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/parameters


- コマンドを実行する（Concourse CI のWebUIへのログイン）

    ```
    # fly --target tutorial login --concourse-url http://localhost:8080
    ```

    - 操作
    - `http://localhost:8080/login?fly_port=43269` でホストOSのブラウザにアクセスし、表示されたtokenを貼り付ける
    - ユーザID: `test`、パスワード: `test` とする
    - 上記操作をすると、Concourse CI のWeb UIにログインできる

        ```
        logging in to team 'main'

        navigate to the following URL in your browser:

        http://localhost:8080/login?fly_port=43269

        or enter token manually (input hidden): 
        target saved
        ```

### 成果物アップロード時のパラメータも設定する

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/parameters#_3

- 以下のコマンド実行

    ```
    # pwd
    /root/concourse-tutorial/tutorials/basic/parameters
    ```

    ```
    # cd ../publishing-outputs
    ```
- `pipeline-parameters.yml`を以下のように作成

    ```
    # cat pipeline-parameters.yml 
    ---
    resources:
    - name: resource-tutorial
        type: git
        source:
        uri: https://github.com/starkandwayne/concourse-tutorial.git
        branch: develop
    - name: resource-gist
        type: git
        source:
        branch: master
        # branch: master
        uri: ((publishing-outputs-gist-uri))
        private_key: ((publishing-outputs-private-key))

    jobs:
    - name: job-bump-date
        serial: true
        plan:
        - get: resource-tutorial
        - get: resource-gist
        - task: bump-timestamp-file
            config:
            platform: linux
            image_resource:
                type: docker-image
                # source: {repository: starkandwayne/concourse}
                source: {repository: getourneau/alpine-bash-git}

            inputs:
                - name: resource-tutorial
                - name: resource-gist
            outputs:
                - name: updated-gist
            run:
                path: resource-tutorial/tutorials/basic/publishing-outputs/bump-timestamp-file.sh
        - put: resource-gist
            params:
            repository: updated-gist
    ```

- `credentials.yml`を以下のように作成

    ```
    # cat credentials.yml 
    publishing-outputs-gist-urii: git@gist.github.com:5e40d8df48fa30baf9abee4b35a2241a.git
    publishing-outputs-private-key: |
        -----BEGIN RSA PRIVATE KEY-----
        MIIEowIBAAKCAQEAp/Tcj0IhF9PG27xq6dH4G72Nqns0KjaFnBABLEZty8DtFiaS
        oESMZgrsrEJe36l1kaE9i/3UWKYrU5LZsDvn8DVc4Y5U0IE/QcmsplH5G0zSA0VJ
        VRLB4higtJV4JPXKyjS1MBG5pLAD7pUbQ+jBj3a6icsHEuSGAfssY2lpSj8UOUwd
        fmnPaaXmyY3x1K1KuaggQ8mLOJNPAJ4x2jar+bap0URlggbu2kFQ4/XakDZVCRxT
        JDybRM37RwrSqLYspnOsaJ18z2EJCuIaiVcySyz7DjVcPTdXQzjkWw8nYPX5IykN
        zHfHC6qgCcN4AJlgFZl1F9xisXnD+B5X5R4LWwIDAQABAoIBAAQc/g3QG8lemV8m
        RSQGzWG4ibCkJcnm3ezNg4nXC7dSuTuypCKiqyGQoO0zDunBV6zCWySDieDF6Qe5
        7/Td8rcyR10KxE7661asHrtQBJ7Did0kpEAeHntwCPeDNZcKIfZDxjAwLvC2ktIT
        +r/2Ak+GI9leDIVM7W88/IBOw5Ja4POYZUpJ+v6ym3kEA+Xgrw5ej0+pOO2NmTFL
        pujQd0P1MnK9kaftRPBg9OlFpq0wjViLDPZKcb9XzHBx6LYHqE2IG7P93b8hMNqP
        A8HAcH8S1h7k3nWT20x/eTVWFTNtGoAsO31nz6QH+E6JbFs6ApNpiON30HkDSFX5
        wnhbsikCgYEA2gCzC01cJSdzp/yZexelM99ivpwEmytE6R4qBemiNAcRwmKTYSeA
        ct3AnxPBj1vIaTgwtq9P/JBEPQMonhTWq3bj1v+70HdBz50njd0Rl58zC9rx6nF4
        2zNOTga3EcmQ2EyeZjxFVSs8v1YtHgT1OUjkcFnS2paBrKkiLhd38WcCgYEAxTsY
        NkoNTQBitORS5wtp39Dr87k6tyVOmRn8MZuBeaKAJ2cIXFHDwRtNm5so+jQS63td
        QVOu4wF9fxBPHrxT4z/D9ASfrmsc1Z7gTRGu/sHGYmdDOomAI6RKBBjNtfgBIekW
        HNrLt+xLXxr8xL7QC8bJtKwPmigWVzo8DWSrme0CgYACrKeFp/lNa2J72Rl47R1V
        uZPYislzreA2i+wwDmGzCbMqE1ODiZyFzDqkuPVS8OlQgSP32ca9bnen1/YTmmXX
        zKmW5aRENnJUPbVShDfHCGjz6Ee3fJTi+4omYua0DSj9vlLjJjIjjVg9cK01BRKN
        FVvYFQIFNHt6xshokFkkWQKBgQCYVS48MDHZuWSDhp4paX1aqwiy8+vPrPbp9VH+
        FreH9OS6ii/A7j4dljL47nxV04aRbnT2keXP20TMsRILETZRnNyCSlfy5TQeIlnn
        7LKWfZ/2PP+F5NGdtbSdOXMZCvYE9PxpSOxzoAQO7s8wPph9oAoGi6Z5UGEA+i+L
        wKdxeQKBgDqFNCOKd50GzA5tavw06AzzsHzYc1PoJBIw3NphYkJimxPbI8RBkkCa
        HsMd73mr7LrLgzZWj7gBD5GXzBqHDdm+/8u4ev3dhquGeBprFJI8E7K7TEi2Spwz
        rm8Hl2L077bV+VOTb177RetIFLt/dY8KREWC72vvqxVIzalhjDnG
        -----END RSA PRIVATE KEY-----
    ```

- 以下のコマンドを実行

- コマンドを実行する（Concourse CI のWebUIへのログイン）

    ```
    # fly --target tutorial login --concourse-url http://localhost:8080
    ```

    - 操作
    - `http://localhost:8080/login?fly_port=43269` でホストOSのブラウザにアクセスし、表示されたtokenを貼り付ける
    - ユーザID: `test`、パスワード: `test` とする
    - 上記操作をすると、Concourse CI のWeb UIにログインできる

        ```
        logging in to team 'main'

        navigate to the following URL in your browser:

        http://localhost:8080/login?fly_port=43269

        or enter token manually (input hidden): 
        target saved
        ```

- パイプラインを削除する

    ```
    # fly -t tutorial destroy-pipeline -p publishing-outputs
    ```

- パイプラインを作成

    ```
    # fly -t tutorial set-pipeline -p publishing-outputs -c pipeline-parameters.yml -l credentials.yml
    ```

- パイプラインのリソースをチェック

    ```
    # fly -t tutorial check-resource -r publishing-outputs/resource-gist
    ```

- パイプラインを有効化
- ※この手順は、オリジナルにはない

    ```
    # fly -t tutorial unpause-pipeline -p publishing-outputs
    ```

- パイプラインを実行し、動作を監視

    ```
    # fly -t tutorial trigger-job -j publishing-outputs/job-bump-date -w
    ```

    - 結果

        ```
        # fly -t tutorial trigger-job -j publishing-outputs/job-bump-date -w
        started publishing-outputs/job-bump-date #1

        selected worker: 971dfbb64a5a
        INFO: found existing resource cache

        selected worker: 971dfbb64a5a
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        Cloning into '/tmp/build/get'...
        45b6011 Bumped date
        initializing
        initializing check: image
        selected worker: 971dfbb64a5a
        selected worker: 971dfbb64a5a
        waiting for docker to come up...
        Pulling getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7...
        docker.io/getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7: Pulling from getourneau/alpine-bash-git
        4fe2ade4980c: Pulling fs layer
        03c196859ec8: Pulling fs layer
        720d2de11875: Pulling fs layer
        4fe2ade4980c: Verifying Checksum
        4fe2ade4980c: Download complete
        03c196859ec8: Verifying Checksum
        03c196859ec8: Download complete
        4fe2ade4980c: Pull complete
        03c196859ec8: Pull complete
        720d2de11875: Verifying Checksum
        720d2de11875: Download complete
        720d2de11875: Pull complete
        Digest: sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7
        Status: Downloaded newer image for getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7
        docker.io/getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7

        Successfully pulled getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7.

        selected worker: 971dfbb64a5a
        running resource-tutorial/tutorials/basic/publishing-outputs/bump-timestamp-file.sh
        + git clone resource-gist updated-gist
        Cloning into 'updated-gist'...
        done.
        + cd updated-gist
        + date
        + echo Sat Sep 2 09:57:59 UTC 2023
        + git config --global user.email nobody@concourse-ci.org
        + git config --global user.name Concourse
        + git add .
        + git commit -m 'Bumped date'
        [main 49d182b] Bumped date
        1 file changed, 1 insertion(+), 1 deletion(-)
        selected worker: 971dfbb64a5a
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        To gist.github.com:5e40d8df48fa30baf9abee4b35a2241a.git
        45b6011..49d182b  HEAD -> main
        selected worker: 971dfbb64a5a
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        Cloning into '/tmp/build/get'...
        49d182b Bumped date
        succeeded
        ```

    - 成功!!

- `credentials.yml`は以下の内容になった

    ```
    # cat credentials.yml 
    publishing-outputs-gist-uri: git@gist.github.com:5e40d8df48fa30baf9abee4b35a2241a.git
    publishing-outputs-private-key: |
        -----BEGIN RSA PRIVATE KEY-----
        MIIEowIBAAKCAQEAp/Tcj0IhF9PG27xq6dH4G72Nqns0KjaFnBABLEZty8DtFiaS
        oESMZgrsrEJe36l1kaE9i/3UWKYrU5LZsDvn8DVc4Y5U0IE/QcmsplH5G0zSA0VJ
        VRLB4higtJV4JPXKyjS1MBG5pLAD7pUbQ+jBj3a6icsHEuSGAfssY2lpSj8UOUwd
        fmnPaaXmyY3x1K1KuaggQ8mLOJNPAJ4x2jar+bap0URlggbu2kFQ4/XakDZVCRxT
        JDybRM37RwrSqLYspnOsaJ18z2EJCuIaiVcySyz7DjVcPTdXQzjkWw8nYPX5IykN
        zHfHC6qgCcN4AJlgFZl1F9xisXnD+B5X5R4LWwIDAQABAoIBAAQc/g3QG8lemV8m
        RSQGzWG4ibCkJcnm3ezNg4nXC7dSuTuypCKiqyGQoO0zDunBV6zCWySDieDF6Qe5
        7/Td8rcyR10KxE7661asHrtQBJ7Did0kpEAeHntwCPeDNZcKIfZDxjAwLvC2ktIT
        +r/2Ak+GI9leDIVM7W88/IBOw5Ja4POYZUpJ+v6ym3kEA+Xgrw5ej0+pOO2NmTFL
        pujQd0P1MnK9kaftRPBg9OlFpq0wjViLDPZKcb9XzHBx6LYHqE2IG7P93b8hMNqP
        A8HAcH8S1h7k3nWT20x/eTVWFTNtGoAsO31nz6QH+E6JbFs6ApNpiON30HkDSFX5
        wnhbsikCgYEA2gCzC01cJSdzp/yZexelM99ivpwEmytE6R4qBemiNAcRwmKTYSeA
        ct3AnxPBj1vIaTgwtq9P/JBEPQMonhTWq3bj1v+70HdBz50njd0Rl58zC9rx6nF4
        2zNOTga3EcmQ2EyeZjxFVSs8v1YtHgT1OUjkcFnS2paBrKkiLhd38WcCgYEAxTsY
        NkoNTQBitORS5wtp39Dr87k6tyVOmRn8MZuBeaKAJ2cIXFHDwRtNm5so+jQS63td
        QVOu4wF9fxBPHrxT4z/D9ASfrmsc1Z7gTRGu/sHGYmdDOomAI6RKBBjNtfgBIekW
        HNrLt+xLXxr8xL7QC8bJtKwPmigWVzo8DWSrme0CgYACrKeFp/lNa2J72Rl47R1V
        uZPYislzreA2i+wwDmGzCbMqE1ODiZyFzDqkuPVS8OlQgSP32ca9bnen1/YTmmXX
        zKmW5aRENnJUPbVShDfHCGjz6Ee3fJTi+4omYua0DSj9vlLjJjIjjVg9cK01BRKN
        FVvYFQIFNHt6xshokFkkWQKBgQCYVS48MDHZuWSDhp4paX1aqwiy8+vPrPbp9VH+
        FreH9OS6ii/A7j4dljL47nxV04aRbnT2keXP20TMsRILETZRnNyCSlfy5TQeIlnn
        7LKWfZ/2PP+F5NGdtbSdOXMZCvYE9PxpSOxzoAQO7s8wPph9oAoGi6Z5UGEA+i+L
        wKdxeQKBgDqFNCOKd50GzA5tavw06AzzsHzYc1PoJBIw3NphYkJimxPbI8RBkkCa
        HsMd73mr7LrLgzZWj7gBD5GXzBqHDdm+/8u4ev3dhquGeBprFJI8E7K7TEi2Spwz
        rm8Hl2L077bV+VOTb177RetIFLt/dY8KREWC72vvqxVIzalhjDnG
        -----END RSA PRIVATE KEY-----
    ```

- `pipeline-parameters.yml`は以下の内容になった

    ```
    # cat pipeline-parameters.yml 
    ---
    resources:
    - name: resource-tutorial
        type: git
        source:
        uri: https://github.com/starkandwayne/concourse-tutorial.git
        branch: develop
    - name: resource-gist
        type: git
        source:
        # branch: master
        branch: main
        uri: ((publishing-outputs-gist-uri))
        private_key: ((publishing-outputs-private-key))

    jobs:
    - name: job-bump-date
        serial: true
        plan:
        - get: resource-tutorial
        - get: resource-gist
        - task: bump-timestamp-file
            config:
            platform: linux
            image_resource:
                type: docker-image
                # source: {repository: starkandwayne/concourse}
                source: {repository: getourneau/alpine-bash-git}

            inputs:
                - name: resource-tutorial
                - name: resource-gist
            outputs:
                - name: updated-gist
            run:
                path: resource-tutorial/tutorials/basic/publishing-outputs/bump-timestamp-file.sh
        - put: resource-gist
            params:
            repository: updated-gist
    ```

### 動的パラメータと秘密パラメータ

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/parameters#_4

- 以下のコマンドを実行する


    ```
    # fly -t tutorial get-pipeline -p parameters
    ```

    - 結果

        ```
        jobs:
        - name: show-animal-names
        plan:
        - config:
            image_resource:
                name: ""
                source:
                repository: busybox
                type: docker-image
            params:
                CAT_NAME: garfield
                DOG_NAME: odie
            platform: linux
            run:
                path: env
            task: show-animal-names
        ```

    ```
    # fly -t tutorial get-pipeline -p publishing-outputs
    ```

    - 結果

        ```
        # fly -t tutorial get-pipeline -p publishing-outputs
        jobs:
        - name: job-bump-date
        plan:
        - get: resource-tutorial
        - get: resource-gist
        - config:
            image_resource:
                name: ""
                source:
                repository: getourneau/alpine-bash-git
                type: docker-image
            inputs:
            - name: resource-tutorial
            - name: resource-gist
            outputs:
            - name: updated-gist
            platform: linux
            run:
                path: resource-tutorial/tutorials/basic/publishing-outputs/bump-timestamp-file.sh
            task: bump-timestamp-file
        - params:
            repository: updated-gist
            put: resource-gist
        serial: true
        resources:
        - name: resource-gist
        source:
            branch: main
            private_key: |
            -----BEGIN RSA PRIVATE KEY-----
            MIIEowIBAAKCAQEAp/Tcj0IhF9PG27xq6dH4G72Nqns0KjaFnBABLEZty8DtFiaS
            oESMZgrsrEJe36l1kaE9i/3UWKYrU5LZsDvn8DVc4Y5U0IE/QcmsplH5G0zSA0VJ
            VRLB4higtJV4JPXKyjS1MBG5pLAD7pUbQ+jBj3a6icsHEuSGAfssY2lpSj8UOUwd
            fmnPaaXmyY3x1K1KuaggQ8mLOJNPAJ4x2jar+bap0URlggbu2kFQ4/XakDZVCRxT
            JDybRM37RwrSqLYspnOsaJ18z2EJCuIaiVcySyz7DjVcPTdXQzjkWw8nYPX5IykN
            zHfHC6qgCcN4AJlgFZl1F9xisXnD+B5X5R4LWwIDAQABAoIBAAQc/g3QG8lemV8m
            RSQGzWG4ibCkJcnm3ezNg4nXC7dSuTuypCKiqyGQoO0zDunBV6zCWySDieDF6Qe5
            7/Td8rcyR10KxE7661asHrtQBJ7Did0kpEAeHntwCPeDNZcKIfZDxjAwLvC2ktIT
            +r/2Ak+GI9leDIVM7W88/IBOw5Ja4POYZUpJ+v6ym3kEA+Xgrw5ej0+pOO2NmTFL
            pujQd0P1MnK9kaftRPBg9OlFpq0wjViLDPZKcb9XzHBx6LYHqE2IG7P93b8hMNqP
            A8HAcH8S1h7k3nWT20x/eTVWFTNtGoAsO31nz6QH+E6JbFs6ApNpiON30HkDSFX5
            wnhbsikCgYEA2gCzC01cJSdzp/yZexelM99ivpwEmytE6R4qBemiNAcRwmKTYSeA
            ct3AnxPBj1vIaTgwtq9P/JBEPQMonhTWq3bj1v+70HdBz50njd0Rl58zC9rx6nF4
            2zNOTga3EcmQ2EyeZjxFVSs8v1YtHgT1OUjkcFnS2paBrKkiLhd38WcCgYEAxTsY
            NkoNTQBitORS5wtp39Dr87k6tyVOmRn8MZuBeaKAJ2cIXFHDwRtNm5so+jQS63td
            QVOu4wF9fxBPHrxT4z/D9ASfrmsc1Z7gTRGu/sHGYmdDOomAI6RKBBjNtfgBIekW
            HNrLt+xLXxr8xL7QC8bJtKwPmigWVzo8DWSrme0CgYACrKeFp/lNa2J72Rl47R1V
            uZPYislzreA2i+wwDmGzCbMqE1ODiZyFzDqkuPVS8OlQgSP32ca9bnen1/YTmmXX
            zKmW5aRENnJUPbVShDfHCGjz6Ee3fJTi+4omYua0DSj9vlLjJjIjjVg9cK01BRKN
            FVvYFQIFNHt6xshokFkkWQKBgQCYVS48MDHZuWSDhp4paX1aqwiy8+vPrPbp9VH+
            FreH9OS6ii/A7j4dljL47nxV04aRbnT2keXP20TMsRILETZRnNyCSlfy5TQeIlnn
            7LKWfZ/2PP+F5NGdtbSdOXMZCvYE9PxpSOxzoAQO7s8wPph9oAoGi6Z5UGEA+i+L
            wKdxeQKBgDqFNCOKd50GzA5tavw06AzzsHzYc1PoJBIw3NphYkJimxPbI8RBkkCa
            HsMd73mr7LrLgzZWj7gBD5GXzBqHDdm+/8u4ev3dhquGeBprFJI8E7K7TEi2Spwz
            rm8Hl2L077bV+VOTb177RetIFLt/dY8KREWC72vvqxVIzalhjDnG
            -----END RSA PRIVATE KEY-----
            uri: git@gist.github.com:5e40d8df48fa30baf9abee4b35a2241a.git
        type: git
        - name: resource-tutorial
        source:
            branch: develop
            uri: https://github.com/starkandwayne/concourse-tutorial.git
        type: git
        ```

    - 秘密鍵が見えてしまう


### リアルなパイプライン - 複数の Job で Resource を共有する

- 以下のサイトを参考に実施

    https://concoursetutorial-ja.site.lkj.io/basics/pipeline-jobs


- 以下のコマンドを実行

    ```
    # cd /root/concourse-tutorial/tutorials/basic/pipeline-jobs/
    ```

- `pipeline.yml`を以下のように修正

    ```
    # cat pipeline.yml
    ---
    resources:
    - name: resource-tutorial
        type: git
        source:
        uri: https://github.com/starkandwayne/concourse-tutorial.git
        branch: develop
    - name: resource-gist
        type: git
        source:
        # branch: master
        branch: main
        uri: ((publishing-outputs-gist-uri))
        private_key: ((publishing-outputs-private-key))

    jobs:
    - name: job-bump-date
        serial: true
        plan:
        - get: resource-tutorial
        - get: resource-gist
        - task: bump-timestamp-file
            config:
            platform: linux
            image_resource:
                type: docker-image
                # source: {repository: starkandwayne/concourse}
                source: {repository: getourneau/alpine-bash-git}
            inputs:
                - name: resource-tutorial
                - name: resource-gist
            outputs:
                - name: updated-gist
            run:
                path: resource-tutorial/tutorials/basic/publishing-outputs/bump-timestamp-file.sh
        - put: resource-gist
            params: {repository: updated-gist}

    - name: job-show-date
        plan:
        - get: resource-tutorial
        - get: resource-gist
            passed: [job-bump-date]
            trigger: true
        - task: show-date
            config:
            platform: linux
            image_resource:
                type: docker-image
                source: {repository: busybox}
            inputs:
                - name: resource-gist
            run:
                path: cat
                args: [resource-gist/bumpme]
    ```

- 以下のコマンドを実行


- コマンドを実行する（Concourse CI のWebUIへのログイン）

    ```
    # fly --target tutorial login --concourse-url http://localhost:8080
    ```

    - 操作
    - `http://localhost:8080/login?fly_port=43269` でホストOSのブラウザにアクセスし、表示されたtokenを貼り付ける
    - ユーザID: `test`、パスワード: `test` とする
    - 上記操作をすると、Concourse CI のWeb UIにログインできる

        ```
        logging in to team 'main'

        navigate to the following URL in your browser:

        http://localhost:8080/login?fly_port=43269

        or enter token manually (input hidden): 
        target saved
        ```

- パイプラインを削除する

    ```
    # fly -t tutorial destroy-pipeline -p publishing-outputs
    ```

- パイプラインを作成

    ```
    # fly -t tutorial set-pipeline -p publishing-outputs -c pipeline.yml -l ../publishing-outputs/credentials.yml
    ```

- パイプラインのリソースをチェック

    ```
    # fly -t tutorial check-resource -r publishing-outputs/resource-gist
    ```

    - 結果

        ```
        checking publishing-outputs/resource-gist in build 2395
        initializing check: resource-gist
        selected worker: 971dfbb64a5a
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        Cloning into '/tmp/git-resource-repo-cache'...
        succeeded
        ```

- パイプラインを有効化
- ※この手順は、オリジナルにはない

    ```
    # fly -t tutorial unpause-pipeline -p publishing-outputs
    ```

    - 結果

        ```
        unpaused 'publishing-outputs'
        ```


- パイプラインを実行し、動作を監視（job-bump-dateを実行し、監視）

    ```
    # fly -t tutorial trigger-job -j publishing-outputs/job-bump-date -w
    ```

    - 結果

        ```
        started publishing-outputs/job-bump-date #1

        selected worker: 971dfbb64a5a
        INFO: found existing resource cache

        selected worker: 971dfbb64a5a
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        Cloning into '/tmp/build/get'...
        49d182b Bumped date
        initializing
        initializing check: image
        selected worker: 971dfbb64a5a
        selected worker: 971dfbb64a5a
        waiting for docker to come up...
        Pulling getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7...
        docker.io/getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7: Pulling from getourneau/alpine-bash-git
        4fe2ade4980c: Pulling fs layer
        03c196859ec8: Pulling fs layer
        720d2de11875: Pulling fs layer
        720d2de11875: Verifying Checksum
        720d2de11875: Download complete
        4fe2ade4980c: Verifying Checksum
        4fe2ade4980c: Download complete
        03c196859ec8: Verifying Checksum
        03c196859ec8: Download complete
        4fe2ade4980c: Pull complete
        03c196859ec8: Pull complete
        720d2de11875: Pull complete
        Digest: sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7
        Status: Downloaded newer image for getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7
        docker.io/getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7

        Successfully pulled getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7.

        selected worker: 971dfbb64a5a
        running resource-tutorial/tutorials/basic/publishing-outputs/bump-timestamp-file.sh
        + git clone resource-gist updated-gist
        Cloning into 'updated-gist'...
        done.
        + cd updated-gist
        + date
        + echo Sat Sep 2 11:15:43 UTC 2023
        + git config --global user.email nobody@concourse-ci.org
        + git config --global user.name Concourse
        + git add .
        + git commit -m 'Bumped date'
        [main 39f8a85] Bumped date
        1 file changed, 1 insertion(+), 1 deletion(-)
        selected worker: 971dfbb64a5a
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        To gist.github.com:5e40d8df48fa30baf9abee4b35a2241a.git
        49d182b..39f8a85  HEAD -> main
        selected worker: 971dfbb64a5a
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        Cloning into '/tmp/build/get'...
        39f8a85 Bumped date
        succeeded
        ```

- 以下のコマンドを実行(job-show-dateの結果を確認)

    ```
    # fly -t tutorial watch -j publishing-outputs/job-show-date
    ```

    - 結果

        ```
        selected worker: 971dfbb64a5a
        INFO: found existing resource cache

        selected worker: 971dfbb64a5a
        INFO: found existing resource cache

        initializing
        initializing check: image
        selected worker: 971dfbb64a5a
        selected worker: 971dfbb64a5a
        INFO: found existing resource cache

        selected worker: 971dfbb64a5a
        running cat resource-gist/bumpme
        Sat Sep 2 11:15:43 UTC 2023
        succeeded
        ```

### 秘密パラメータを資格情報マネージャで管理する

- 以下のサイト
    https://concoursetutorial-ja.site.lkj.io/basics/secret-parameters

- 資格情報マネージャは、Vault を使用するので、ここはとりあえず飛ばし


### Miscellaneous（その他）

- 以下のサイト

    https://concoursetutorial-ja.site.lkj.io/miscellaneous/

- 秘密パラメータを資格情報マネージャで管理するの`bucc`は実行していない。
- このため、 レッスンには、`fly -t bucc` コマンドでなく、`fly -t tutorial`コマンドを使用
- パラメータの渡しは、`-v` または `-l` フラグで `fly set-pipeline`` を使ってコマンドラインを使用


### デプロイの前にテストを実行する

- 以下のサイト

    https://concoursetutorial-ja.site.lkj.io/miscellaneous/run-tests-before-deploy

- 2023-09-03、 Cloud Foundry のアカウントを作成したいないため、飛ばし

### Docker イメージの作成・利用

- 以下のサイトを参考にして実行

    https://concoursetutorial-ja.site.lkj.io/miscellaneous/docker-images


- credentials.yml の更新

- 以下のコマンドを実行し、`credentials.yml` の更新内容を確認

    ```
    # cd ~/concourse-install/
    ```

    ```
    # cat credentials.yml 
    publishing-outputs-gist-uri: git@gist.github.com:5e40d8df48fa30baf9abee4b35a2241a.git
    publishing-outputs-private-key: |
        -----BEGIN RSA PRIVATE KEY-----
        MIIEowIBAAKCAQEAp/Tcj0IhF9PG27xq6dH4G72Nqns0KjaFnBABLEZty8DtFiaS
        oESMZgrsrEJe36l1kaE9i/3UWKYrU5LZsDvn8DVc4Y5U0IE/QcmsplH5G0zSA0VJ
        VRLB4higtJV4JPXKyjS1MBG5pLAD7pUbQ+jBj3a6icsHEuSGAfssY2lpSj8UOUwd
        fmnPaaXmyY3x1K1KuaggQ8mLOJNPAJ4x2jar+bap0URlggbu2kFQ4/XakDZVCRxT
        JDybRM37RwrSqLYspnOsaJ18z2EJCuIaiVcySyz7DjVcPTdXQzjkWw8nYPX5IykN
        zHfHC6qgCcN4AJlgFZl1F9xisXnD+B5X5R4LWwIDAQABAoIBAAQc/g3QG8lemV8m
        RSQGzWG4ibCkJcnm3ezNg4nXC7dSuTuypCKiqyGQoO0zDunBV6zCWySDieDF6Qe5
        7/Td8rcyR10KxE7661asHrtQBJ7Did0kpEAeHntwCPeDNZcKIfZDxjAwLvC2ktIT
        +r/2Ak+GI9leDIVM7W88/IBOw5Ja4POYZUpJ+v6ym3kEA+Xgrw5ej0+pOO2NmTFL
        pujQd0P1MnK9kaftRPBg9OlFpq0wjViLDPZKcb9XzHBx6LYHqE2IG7P93b8hMNqP
        A8HAcH8S1h7k3nWT20x/eTVWFTNtGoAsO31nz6QH+E6JbFs6ApNpiON30HkDSFX5
        wnhbsikCgYEA2gCzC01cJSdzp/yZexelM99ivpwEmytE6R4qBemiNAcRwmKTYSeA
        ct3AnxPBj1vIaTgwtq9P/JBEPQMonhTWq3bj1v+70HdBz50njd0Rl58zC9rx6nF4
        2zNOTga3EcmQ2EyeZjxFVSs8v1YtHgT1OUjkcFnS2paBrKkiLhd38WcCgYEAxTsY
        NkoNTQBitORS5wtp39Dr87k6tyVOmRn8MZuBeaKAJ2cIXFHDwRtNm5so+jQS63td
        QVOu4wF9fxBPHrxT4z/D9ASfrmsc1Z7gTRGu/sHGYmdDOomAI6RKBBjNtfgBIekW
        HNrLt+xLXxr8xL7QC8bJtKwPmigWVzo8DWSrme0CgYACrKeFp/lNa2J72Rl47R1V
        uZPYislzreA2i+wwDmGzCbMqE1ODiZyFzDqkuPVS8OlQgSP32ca9bnen1/YTmmXX
        zKmW5aRENnJUPbVShDfHCGjz6Ee3fJTi+4omYua0DSj9vlLjJjIjjVg9cK01BRKN
        FVvYFQIFNHt6xshokFkkWQKBgQCYVS48MDHZuWSDhp4paX1aqwiy8+vPrPbp9VH+
        FreH9OS6ii/A7j4dljL47nxV04aRbnT2keXP20TMsRILETZRnNyCSlfy5TQeIlnn
        7LKWfZ/2PP+F5NGdtbSdOXMZCvYE9PxpSOxzoAQO7s8wPph9oAoGi6Z5UGEA+i+L
        wKdxeQKBgDqFNCOKd50GzA5tavw06AzzsHzYc1PoJBIw3NphYkJimxPbI8RBkkCa
        HsMd73mr7LrLgzZWj7gBD5GXzBqHDdm+/8u4ev3dhquGeBprFJI8E7K7TEi2Spwz
        rm8Hl2L077bV+VOTb177RetIFLt/dY8KREWC72vvqxVIzalhjDnG
        -----END RSA PRIVATE KEY-----

    docker-hub-email: moriyama.kazuhiro@earthsys-lab.co.jp
    docker-hub-username: kuzumusen
    docker-hub-password: z@yaNa053
    ```

- 以下のコマンドを実行し、`pipeline.yml`の内容を確認

    ```
    # cd ~/concourse-tutorial/tutorials/miscellaneous/docker-images/
    ```

    ```
    # cat pipeline.yml 
    ---
    resources:
    - name: tutorial
        type: git
        source:
        uri: https://github.com/drnic/concourse-tutorial.git
        branch: develop

    - name: hello-world-docker-image
        type: docker-image
        source:
        email: ((docker-hub-email))
        username: ((docker-hub-username))
        password: ((docker-hub-password))
        repository: ((docker-hub-username))/concourse-tutorial-hello-world

    jobs:
    - name: publish
        public: true
        plan:
        - get: tutorial
        - put: hello-world-docker-image
            params:
            build: tutorial/tutorials/miscellaneous/docker-images/docker
        - task: run
            config:
            platform: linux
            image_resource:
                type: docker-image
                source:
                repository: ((docker-hub-username))/concourse-tutorial-hello-world
            run:
                path: /bin/hello-world
                args: []
            params:
                NAME: ((docker-hub-username))
    ```

- コマンドを実行する（Concourse CI のWebUIへのログイン）

    ```
    # fly --target tutorial login --concourse-url http://localhost:8080
    ```

    - 操作
    - `http://localhost:8080/login?fly_port=43269` でホストOSのブラウザにアクセスし、表示されたtokenを貼り付ける
    - ユーザID: `test`、パスワード: `test` とする
    - 上記操作をすると、Concourse CI のWeb UIにログインできる

        ```
        logging in to team 'main'

        navigate to the following URL in your browser:

        http://localhost:8080/login?fly_port=43269

        or enter token manually (input hidden): 
        target saved
        ```

- パイプラインを削除する

    ```
    # fly -t tutorial destroy-pipeline -p push-docker-image
    ```

-　以下のコマンドを実行

    ```
    # fly -t tutorial set-pipeline -p push-docker-image -c pipeline.yml -n -l ~/concourse-install/credentials.yml
    ```

    - ※補足

        ```
        -n オプションを付けると、`apply configuration? [yN]: y` といった問合せがなくなる

        ＜ChatGPTより＞
        Concourse CI/CDのfly set-pipelineコマンドにおいて、-nオプションは "non-interactive" モードを意味します。このオプションを使用することで、対話的なプロンプトなしでパイプラインを設定することができます。

        通常、fly set-pipelineコマンドは対話モードで動作し、設定情報をユーザーに対話的に尋ねます。しかし、-nオプションを指定することで、コマンドの実行中にユーザーからの入力を待たずに、事前に指定した設定ファイル (-c pipeline.yml で指定されたファイル) を使用してパイプラインを設定します。

        このように、-nオプションを使うことで、自動化されたスクリプトやバッチジョブなどでConcourseパイプラインを設定する際に役立ちます。
        ```

    - 問合せが必要ならば、以下のコマンドとする

        ```
        # fly -t tutorial set-pipeline -p push-docker-image -c pipeline.yml -l ~/concourse-install/credentials.yml
        ```

<del>

- パイプラインのリソースをチェック

    ```
    # fly -t tutorial check-resource -r publishing-outputs/resource-gist
    ```

    - 結果

        ```
        # fly -t tutorial check-resource -r publishing-outputs/resource-gist
        checking publishing-outputs/resource-gist in build 4403
        initializing check: resource-gist
        selected worker: 971dfbb64a5a
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        HEAD is now at 39f8a85 Bumped date
        succeeded
        ```

</del>

    ```
    # fly -t tutorial unpause-pipeline -p push-docker-image
    ```

    ```
    # fly -t tutorial trigger-job -j push-docker-image/publish -w
    ```

    - 結果

        ```
        started push-docker-image/publish #1

        selected worker: 971dfbb64a5a
        Cloning into '/tmp/build/get'...
        a3edcb3 restrict mkdocs packages until can make time to upgrade https://ci2.starkandwayne.com/teams/starkandwayne/pipelines/concourse-tutorial/jobs/website-master/builds/32
        selected worker: 971dfbb64a5a
        waiting for docker to come up...
        WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
        Configure a credential helper to remove this warning. See
        https://docs.docker.com/engine/reference/commandline/login/#credentials-store

        Login Succeeded
        DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
                    BuildKit is currently disabled; enable it by removing the DOCKER_BUILDKIT=0
                    environment-variable.

        Sending build context to Docker daemon  3.072kB
        Step 1/4 : FROM busybox
        latest: Pulling from library/busybox
        3f4d90098f5b: Pulling fs layer
        3f4d90098f5b: Verifying Checksum
        3f4d90098f5b: Download complete
        3f4d90098f5b: Pull complete
        Digest: sha256:3fbc632167424a6d997e74f52b878d7cc478225cffac6bc977eedfe51c7f4e79
        Status: Downloaded newer image for busybox:latest
        ---> a416a98b71e2
        Step 2/4 : ADD hello-world /bin/hello-world
        ---> 9275d00a40f1
        Step 3/4 : ENV NAME=world
        ---> Running in e574eb8d20ac
        Removing intermediate container e574eb8d20ac
        ---> 6df0b0183dfb
        Step 4/4 : ENTRYPOINT ["/bin/hello-world"]
        ---> Running in f0ae48c1e62c
        Removing intermediate container f0ae48c1e62c
        ---> 09a86b0b13ee
        Successfully built 09a86b0b13ee
        Successfully tagged kuzumusen/concourse-tutorial-hello-world:latest
        The push refers to repository [docker.io/kuzumusen/concourse-tutorial-hello-world]
        4ed197c16584: Preparing
        3d24ee258efc: Preparing
        3d24ee258efc: Mounted from library/busybox
        4ed197c16584: Pushed
        latest: digest: sha256:727b34b02e7f858aa93d9efc21eff99da0d9cabcd4fbd1885e78a0065bf8beb5 size: 735
        selected worker: 971dfbb64a5a
        waiting for docker to come up...
        WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
        Configure a credential helper to remove this warning. See
        https://docs.docker.com/engine/reference/commandline/login/#credentials-store

        Login Succeeded
        Pulling kuzumusen/concourse-tutorial-hello-world@sha256:727b34b02e7f858aa93d9efc21eff99da0d9cabcd4fbd1885e78a0065bf8beb5...
        docker.io/kuzumusen/concourse-tutorial-hello-world@sha256:727b34b02e7f858aa93d9efc21eff99da0d9cabcd4fbd1885e78a0065bf8beb5: Pulling from kuzumusen/concourse-tutorial-hello-world
        3f4d90098f5b: Pulling fs layer
        20b2426c6bbd: Pulling fs layer
        20b2426c6bbd: Verifying Checksum
        20b2426c6bbd: Download complete
        3f4d90098f5b: Verifying Checksum
        3f4d90098f5b: Download complete
        3f4d90098f5b: Pull complete
        20b2426c6bbd: Pull complete
        Digest: sha256:727b34b02e7f858aa93d9efc21eff99da0d9cabcd4fbd1885e78a0065bf8beb5
        Status: Downloaded newer image for kuzumusen/concourse-tutorial-hello-world@sha256:727b34b02e7f858aa93d9efc21eff99da0d9cabcd4fbd1885e78a0065bf8beb5
        docker.io/kuzumusen/concourse-tutorial-hello-world@sha256:727b34b02e7f858aa93d9efc21eff99da0d9cabcd4fbd1885e78a0065bf8beb5

        Successfully pulled kuzumusen/concourse-tutorial-hello-world@sha256:727b34b02e7f858aa93d9efc21eff99da0d9cabcd4fbd1885e78a0065bf8beb5.

        initializing
        initializing check: image
        selected worker: 971dfbb64a5a
        selected worker: 971dfbb64a5a
        waiting for docker to come up...
        Pulling kuzumusen/concourse-tutorial-hello-world@sha256:727b34b02e7f858aa93d9efc21eff99da0d9cabcd4fbd1885e78a0065bf8beb5...
        docker.io/kuzumusen/concourse-tutorial-hello-world@sha256:727b34b02e7f858aa93d9efc21eff99da0d9cabcd4fbd1885e78a0065bf8beb5: Pulling from kuzumusen/concourse-tutorial-hello-world
        3f4d90098f5b: Pulling fs layer
        20b2426c6bbd: Pulling fs layer
        3f4d90098f5b: Verifying Checksum
        3f4d90098f5b: Download complete
        20b2426c6bbd: Verifying Checksum
        20b2426c6bbd: Download complete
        3f4d90098f5b: Pull complete
        20b2426c6bbd: Pull complete
        Digest: sha256:727b34b02e7f858aa93d9efc21eff99da0d9cabcd4fbd1885e78a0065bf8beb5
        Status: Downloaded newer image for kuzumusen/concourse-tutorial-hello-world@sha256:727b34b02e7f858aa93d9efc21eff99da0d9cabcd4fbd1885e78a0065bf8beb5
        docker.io/kuzumusen/concourse-tutorial-hello-world@sha256:727b34b02e7f858aa93d9efc21eff99da0d9cabcd4fbd1885e78a0065bf8beb5

        Successfully pulled kuzumusen/concourse-tutorial-hello-world@sha256:727b34b02e7f858aa93d9efc21eff99da0d9cabcd4fbd1885e78a0065bf8beb5.

        selected worker: 971dfbb64a5a
        running /bin/hello-world
        hello kuzumusen
        succeeded
        ```

### バージョニングとビルド番号

- 以下のサイトが参考

    https://concoursetutorial-ja.site.lkj.io/miscellaneous/versions-and-buildnumbers

- 2023-09-03、AWSのアカウントがまだないこと、資格情報マネージャ（Credhub）を使用していないことから、飛ばし


### Github Release を input にする

- 以下のサイトを参考にしている

    https://concoursetutorial-ja.site.lkj.io/miscellaneous/github-release-input#github-release-input


- 以下のコマンドを実行

    ```
    # cd ~/concourse-tutorial/tutorials/miscellaneous/github-release-input/
    ```

- コマンドを実行する（Concourse CI のWebUIへのログイン）

    ```
    # fly --target tutorial login --concourse-url http://localhost:8080
    ```

    - 操作
    - `http://localhost:8080/login?fly_port=43269` でホストOSのブラウザにアクセスし、表示されたtokenを貼り付ける
    - ユーザID: `test`、パスワード: `test` とする
    - 上記操作をすると、Concourse CI のWeb UIにログインできる

        ```
        logging in to team 'main'

        navigate to the following URL in your browser:

        http://localhost:8080/login?fly_port=43269

        or enter token manually (input hidden): 
        target saved
        ```

- パイプラインを削除する

    ```
    # fly -t tutorial destroy-pipeline -p github-release-input -n
    ```

-　以下のコマンドを実行

    ```
    # fly -t tutorial set-pipeline -p github-release-input -c pipeline.yml -n 
    ```

- パイプラインのリソースをチェック

    ```
    # fly -t tutorial check-resource -r github-release-input/github-release-shield
    ```

    - 結果

        ```
        ```

    ```
    # fly -t tutorial unpause-pipeline -p github-release-input
    ```

    ```
    # fly -t tutorial trigger-job -j github-release-input/shield -w
    ```

    - 結果

        ```
        started github-release-input/shield #1

        selected worker: 971dfbb64a5a
        downloading asset: shield-darwin-amd64
        downloading asset: shield-darwin-arm64
        downloading asset: shield-linux-amd64
        downloading asset: shield-server-linux-amd64.tar.gz
        initializing
        initializing check: image
        selected worker: 971dfbb64a5a
        selected worker: 971dfbb64a5a
        waiting for docker to come up...
        Pulling ubuntu@sha256:aabed3296a3d45cede1dc866a24476c4d7e093aa806263c27ddaadbdce3c1054...
        docker.io/library/ubuntu@sha256:aabed3296a3d45cede1dc866a24476c4d7e093aa806263c27ddaadbdce3c1054: Pulling from library/ubuntu
        445a6a12be2b: Pulling fs layer
        445a6a12be2b: Verifying Checksum
        445a6a12be2b: Download complete
        445a6a12be2b: Pull complete
        Digest: sha256:aabed3296a3d45cede1dc866a24476c4d7e093aa806263c27ddaadbdce3c1054
        Status: Downloaded newer image for ubuntu@sha256:aabed3296a3d45cede1dc866a24476c4d7e093aa806263c27ddaadbdce3c1054
        docker.io/library/ubuntu@sha256:aabed3296a3d45cede1dc866a24476c4d7e093aa806263c27ddaadbdce3c1054

        Successfully pulled ubuntu@sha256:aabed3296a3d45cede1dc866a24476c4d7e093aa806263c27ddaadbdce3c1054.

        selected worker: 971dfbb64a5a
        running ls -opR .
        .:
        total 0
        drwxr-xr-x. 1 root 6 Sep  3 06:49 github-release-shield/

        ./github-release-shield:
        total 155276
        -rw-r--r--. 1 root       613 Sep  3 06:47 body
        -rw-r--r--. 1 root        40 Sep  3 06:47 commit_sha
        -rw-r--r--. 1 root   8842352 Sep  3 06:48 shield-darwin-amd64
        -rw-r--r--. 1 root   8601890 Sep  3 06:48 shield-darwin-arm64
        -rw-r--r--. 1 root   9013177 Sep  3 06:48 shield-linux-amd64
        -rw-r--r--. 1 root 132510067 Sep  3 06:49 shield-server-linux-amd64.tar.gz
        -rw-r--r--. 1 root         6 Sep  3 06:47 tag
        -rw-r--r--. 1 root        20 Sep  3 06:47 timestamp
        -rw-r--r--. 1 root        59 Sep  3 06:47 url
        -rw-r--r--. 1 root         5 Sep  3 06:47 version
        initializing
        initializing check: image
        selected worker: 971dfbb64a5a
        INFO: found existing resource cache

        selected worker: 971dfbb64a5a
        running cat github-release-shield/version
        8.8.6succeeded
        ```