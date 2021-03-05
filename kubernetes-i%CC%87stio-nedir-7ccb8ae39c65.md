
# Kubernetes İstio nedir?

Istio

Ağustos ayı içerisinde [Istanbul Coders](https://www.meetup.com/tr-TR/Istanbul-Hackers/)’ın düzenlemiş olduğu ve [Taha Özket](https://www.linkedin.com/in/tahaozket) ‘in değerli sunumunu paylaştığı meetup’a katıldım. Taha, bizlere Google tarafından geliştirilerek açık kaynak komünitesine(***Cloud Native Computing Foundation*)** devredilen Kubernetes projesinin geçtiğimiz dört yılda geldiği noktayı ve bugün container-orchestration konusunda çözdüğü problemleri paylaştı.Aynı zamanda bizlere geçtiğimiz haftalarda **Google Cloud Next ’18** etkinliğinde duyurulan **Kubernetes Istio** ‘ yu bir demo ile gösterdi.

## Peki nedir bu Istio?

Kendi araştırmalarım doğrultusunda,Istio’yu,Kubernetes’in üzerine deploy edilen bir uygulama olarak düşünebilirsiniz. Servisleri bağlamanıza, güvenli hale getirmenize, kontrol etmenize ve izlemenize,geçen trafiği istediğiniz yere yönlendirmenize izin verir.

![Şekil-1](https://cdn-images-1.medium.com/max/2534/1*Xm9AL_hq3EgDOhdPe-tnNw.png)*Şekil-1*

![Şekil-2](https://cdn-images-1.medium.com/max/8064/1*k_yxUeu8VLr7nxtBcrMSwA@2x.jpeg)*Şekil-2*

Istıo’yu Kubernetes Cluster’ınıza deploy ettiğinizde aslında mevcut pod’unuzn içinde, virtual service olarak bir container daha ayağa kalkıyor.(Bknz Şekil-2)Mevcut container’ınıza gelen ve giden trafik ilk önce bu sidecar envoy proxy container’ı üzerinden geçmeye başlıyor. Envoy proxy, bütün servis mash katmanı içindeki servislerin ,gelen ve giden trafiğinde araya giren , C++ ile yazılmış,üstün performanslı bir open source üründür.Envoy proxy ile ilgili detaylı bilgiyi [**bu adres](https://www.envoyproxy.io)** üzerinde bulabilirsiniz.

## **Service Mesh nedir?**

Microservice mimarisindeki uygulamalarınız için tasarlanmış konfigure edilebilen bir infrastructure katmanıdır.Bu sayede uygulamalar daha güvenilir daha esnek ve daha hızlı şekilde birbiri ile haberleşebiliyor. Service Mesh aynı zamanda size service discovery,(Consul ve etcd gibi),load balance,encryption, authentication ve authorization gibi özellikleri de sağlıyor.Service Mesh’in boyutu ve karmaşıklığı arttıkça bu katman içinde çalışan servisleri anlaması ve yönetmesi zorlaşıyor.Fazlasıyla karmaşık bir operasyonel iş gerektiriyor. **Kubernetes Istio** burda devreye giriyor.Operasyonel kontrolu ele alıp microservice uygulamalarının gereksinimlerini karşılamak üzere eksiksiz bir çözüm sunuyor.

Şimdilik dünyanın önde gelen Cloud firmalarından sadece** Google Cloud Platform **bunu bir [hizmet](https://cloud.google.com/istio/) olarak sunuyor.Yakın zamanda AWS ve Azure tarafınında bu özelliği , mevcut Container servislerine katacağına eminim.Hatta **Redhat Openshift Container Platform 4 **versiyonunda entegre bir şekilde geliyor.

Istio’yu denemek isteyenler için katacoda güzel bir lab hazırlamış. [**Buradan](https://www.katacoda.com/courses/istio/deploy-istio-on-kubernetes)** ulaşabilirsiniz.

Istio Kurulumunu alttaki makalemde bulabilirsiniz.
[**Kubernetes Istio Kurulumu**
*Daha önce Istio hakkında bilgilendirme amaçlı bir makale yazmıştım.Okumak isteyenler için altta linkini bırakıyorum.*medium.com](https://medium.com/devopsturkiye/https-medium-com-emreozkan-kubernetes-istio-kurulumu-777d1d3840dc)

Diğer makalelerimi okumak için profilimi ziyaret edebilirsiniz.
[**Emre Özkan ☁️ 🐧 🐳 ☸️ - Medium**
*Read writing from Emre Özkan ☁️ 🐧 🐳 ☸️ on Medium. @Linux System Engineer @Devops Fancier https://sysaix.com ☁️ 🐧…*medium.com](https://medium.com/@emreozkan)

Bloguma aşağıdaki linkten ulaşabilirsiniz.
[**Cloud Devops Unix Linux Container Tutorial**
*Edit description*sysaix.com](https://sysaix.com)
