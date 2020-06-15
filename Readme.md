Kubernetes-in-CentOS7
=
這篇文章，是我第一次使用並實裝Kubernetes在我的電腦上，老實說我完全不了解它，因此我找了許多資料。
基本上是照著[參考網站](https://blog.tomy168.com/2019/08/centos-76-kubernetes.html) 做的，但是中途有好多問題所以我就上網一一解惑了。

到/etc/hsots內更改，hosts裡有什麼??
-
透過 hosts 設定檔手動設定主機名稱與 IP 位址是系統與網站管理者常用的除錯技巧。

對於網際網路的基礎知識有了解的人應該都清楚網址與 IP 的對應關係，每個網站的網址都會對應一個或多個 IP 位址，
當使用者要連上一個網站之前，要先知道網站的網址（如 blog.gtwang.org），
接著連線至 DNS 伺服器，查詢該網址所對應的 IP 位址，獲得網站的實際 IP 位址之後，才能連上該網站瀏覽上面的內容。

<p align="center">
<img src ="https://github.com/TKTim/Kubernetes-in-CentOS7/blob/master/windows-hosts-file-configuration-9.png" width=375 >
</p>



禁用Swap，什麼是 Swap 空間？
-
在 Linux 上的 『Swap 空間』 是在當實體記憶體(RAM)用完時才會使用到，假如系統需要更多的記憶體資源，而實體記憶體已經用完，
記憶體上不活動的頁面將會被移到 swap 空間。
雖然 swap 空間可以幫助系統增加一小部份容量的 RAM，不過不能將它當作更多記憶體的替代品。 Swap 空間是位於硬碟上，它的存取速度比起實體記憶體慢了很多。


禁用Selinux
-
#vi /etc/selinux/config
SELinux（Security-Enhanced Linux）是為了避免使用者資源的誤用，
在進行程序、檔案等細部權限設定依據的一個核心模組！遵從最小權限理念，預設情況下所有程序都是被拒絕的。

SELinux 共有三種模式如下：

```
Enforcing：強制模式，依據設定來限制檔案資源存取。
Permissive：寬容模式，不限制檔案資源存取，但仍會依據設定檢查並記錄相關訊息。
Disabled：停用模式，SELinux 已被停用。
```

iptables 是甚麼?
-


iptables 設定時主要分三種類型，分別是 INPUT, FORWARD 及 OUTPUT。

```
INPUT: 這個行為是外來的連線，例如從遠端 SSH 到伺服器，iptables 會將這個連線定義為 INPUT。
FORWARD: 這是外來的連線，但最終目的地不是伺服器本身，只是轉送到其他機器，例如路由器，除了 routing, NAT 等網路服務外，一般上不會用到這種類型。
OUTPUT: 從本機連接到外部的連線，這個可以允許或阻擋本機連接對外的服務。
```

而 iptables 對匹配連線會有三種不同動作：

```
Accept： 允許連線。
Drop： 將連接 Drop 掉，就像連線從未建立，不會反回錯誤。
Reject：不允許連線，但會反回錯誤。
```

啟用 br_netfilter，
-
#modprobe br_netfilter
可憐的我，兩個指令都不知道什麼意思，只好爬文囉!!!

先來看看modprobe:
```
modprobe命令用於智能地向內核中加載模塊或者從內核中移除模塊，可載入指定的個別模塊，或是載入一組相依的模塊。
```
想查更多資料，可以看這裡: [參考網站](https://man.linuxde.net/modprobe)

再來看看 br_netfilter:

```
透明防火牆(Transparent Firewall)又稱橋接模式防火牆（Bridge Firewall）。
簡單來說，就是在網橋設備上加入防火牆功能。透明防火牆具有部署能力強、隱蔽性好、安全性高的優點。
```

現在來說說，sysctl -p 到底在幹嘛?
-
我直接在cmd裡輸入 sysctl -h ，叫出help囉!
<p align="center">
<img src ="https://github.com/TKTim/Kubernetes-in-CentOS7/blob/master/5.png" width=375 >
</p>

lsmod，ls指令還有mod!?
-
lsmod命令用於顯示已經加載到內核中的模塊的狀態信息。執行lsmod命令後會列出所有已載入系統的模塊。 Linux操作系統的核心具有模塊化的特性，
應此在編譯核心時，務須把全部的功能都放入核心。您可以將這些功能編譯成一個個單獨的模塊，待需要時再分別載入。
```
[root@server ~]# lsmod | grep br_netfilter
br_netfilter           22256  0 
bridge                151336  1 br_netfilter
```

雖然跟 yum 是老朋友，但我們來了解一下。
-
```
yum clean
#清除安裝下載時的暫套件原始檔，大多是存放在/var/cache/yum，通常會下yum clean packages或是yum clean all，一次全刪除。
```
```
yum repolist 查詢目前CentOS 主機軟體套件庫中所支援的套件數量。
```
<p align="center">
<img src ="https://github.com/TKTim/Kubernetes-in-CentOS7/blob/master/6.png" width=700 >
</p>

安裝docker-ce和kubernetes，What are those!?
-

docker-ce
-

<p align="center">
<img src ="https://github.com/TKTim/Kubernetes-in-CentOS7/blob/master/7.png" width=400 >
</p>


Docker是一種輕量級的作業系統虛擬化解決方案，相較於傳統在Host作業系統上安裝Guest作業系統的硬體虛擬化方式
，Docker可以直接在同一個Host作業系統核心上，以「容器」來區分應用程式的執行環境，也就是直接在系統層上完成虛擬化。
因此Docker執行程式的效率通常會比傳統虛擬化的方式還要來得好，可以節省許多硬體資源。在實務上，Docker常被用來部署資料庫、
Web應用程式等伺服器相關的程式，因為只要設定好執行環境，再將映像檔保存下來之後
，就可以一直重複使用。對於程式開發人員來說，Docker也可以用來模擬不同環境下，程式是否能正常編譯和執行。

[我的網站](https://timleesdailyfactory.blogspot.com/2020/03/linux-note-0325-centos-7-ssl-docker.html)裡有補充一些資訊，可以看看。

kubernetes
-
<p align="center">
<img src ="https://github.com/TKTim/Kubernetes-in-CentOS7/blob/master/0_5N7SlevIHOdKB-yC.jpg" width=500 >
</p>

這裡有篇精彩的文章，我就是從[這裡](https://medium.com/@C.W.Hu/kubernetes-basic-concept-tutorial-e033e3504ec0)學會的，看看吧!

這是做完nginx的樣子:
<p align="center">
<img src ="https://github.com/TKTim/Kubernetes-in-CentOS7/blob/master/8.png" width=500 >
</p>
喔!等等我還沒解釋，甚麼是nginx。

什麼是nginx
-
你知道嗎?我就不搶走別人的功勞了，看看這個網站

[我是這個網站，點我](https://ithelp.ithome.com.tw/articles/10196261)。
[關於kubeadm init](https://k8smeetup.github.io/docs/admin/kubeadm/)

```
#-kudectl get nodes 
查詢nodes
#kubectl get pod --all-namespaces
依照namespaces顯示所有pod

```

YAML
-
在實作kubernetes的應用時，會時常使用到yaml檔案來create，我們來了解一下他是做甚麼的吧。

```
YAML的語法和其他高階語言類似，並且可以簡單表達清單、雜湊表，純量等資料形態。它使用空白符號縮排和大量依賴外觀的特色，特別適合用來表達或編輯資料結構、各種設定檔、傾印除錯內容、檔案大綱（例如：許多電子郵件標題格式和YAML非常接近）。儘管它比較適合用來表達階層式（hierarchical model）的資料結構，不過也有精緻的語法可以表示關聯性（relational model）的資料。[5]由於YAML使用空白字元和分行來分隔資料，使得它特別適合用grep／Python／Perl／Ruby操作。其讓人最容易上手的特色是巧妙避開各種封閉符號，如：引號、各種括號等，這些符號在巢狀結構時會變得複雜而難以辨認。
```
看看人家  [維基](https://zh.wikipedia.org/wiki/YAML#%E8%AA%9E%E8%A8%80%E7%9A%84%E6%A7%8B%E6%88%90%E5%85%83%E7%B4%A0)  寫得多好。

新增dashboard功能:
-
我高興到落淚，如果你不知道我落淚的原因，我卡在dashboard卡了快一個禮拜，但我終於OK了。
```
在瀏覽器輸入:
<IP address>:<Port> (教學裡都有)
但是!但是!! 問題來了 Client sent an HTTP request to an HTTPS server.
其實只要把前面預設的，http改成https就好了。
全文:
https://<IP address>:<Port>
```

解剖pod身分證，yaml檔案
=
```
apiVersion: v1
 kind: Pod
 metadata:
   name: kubernetes-demo-pod
   labels:
     app: demoApp
 spec:
   containers:
     - name: kubernetes-demo-container
       image: hcwxd/kubernetes-demo
       ports:
         - containerPort: 3000
```

### apiVersion
```
該元件的版本號
```

### kind
```
該元件是什麼屬性，常見有 Pod、Node、Service、Namespace、ReplicationController 等
```
### metadata
```
name
指定該 Pod 的名稱
labels
指定該 Pod 的標籤，這裡我們暫時幫它上標籤為 app: demoApp
```
### spec
```
。container.name
指定運行出的 Container 的名稱
。container.image
指定 Container 要使用哪個 Image，這裡會從 DockerHub 上搜尋
。container.ports
指定該 Container 有哪些 port number 是允許外部資源存取
```
<p style="color:blue;">本章取自於:<a href="https://medium.com/@C.W.Hu/kubernetes-implement-ingress-deployment-tutorial-7431c5f96c3e " style="color:blue;">Kubernetes 基礎教學 </a></p>





















