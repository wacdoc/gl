# Crea o teu propio servidor de envío de correo SMTP

## preámbulo

SMTP pode comprar directamente servizos de provedores de nube, como:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Ali cloud push de correo electrónico](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Tamén podes crear o teu propio servidor de correo: envío ilimitado, baixo custo global.

A continuación, demostramos paso a paso como construír o noso propio servidor de correo.

## Selección do servidor

O servidor SMTP autoaloxado require unha IP pública cos portos 25, 456 e 587 abertos.

As nubes públicas de uso habitual bloquearon estes portos de forma predeterminada e é posible abrilos emitindo unha orde de traballo, pero despois de todo é moi problemático.

Recomendo mercar nun servidor que teña estes portos abertos e admita a configuración de nomes de dominio inversos.

Aquí, recomendo [Contabo](https://contabo.com) .

Contabo é un provedor de hospedaxe con sede en Múnic, Alemaña, fundado en 2003 con prezos moi competitivos.

Se escolle o euro como moeda de compra, o prezo será máis barato (un servidor con 8 GB de memoria e 4 CPU custa uns 529 yuans ao ano, e a tarifa de instalación inicial é gratuíta durante un ano).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

Ao facer un pedido, comente que `prefer AMD` , e o servidor con CPU AMD terá un mellor rendemento.

A continuación, tomarei o VPS de Contabo como exemplo para demostrar como construír o teu propio servidor de correo.

## Configuración do sistema Ubuntu

O sistema operativo aquí é Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Se o servidor en ssh mostra `Welcome to TinyCore 13!` (como se mostra na figura seguinte), significa que o sistema aínda non se instalou. Desconecte ssh e agarde uns minutos para iniciar sesión de novo.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Cando apareza `Welcome to Ubuntu 22.04.1 LTS` , a inicialización está completa e pode continuar cos seguintes pasos.

### [Opcional] Inicializa o ambiente de desenvolvemento

Este paso é opcional.

Para comodidade, poño a instalación e configuración do sistema do software de ubuntu en [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

Executa o seguinte comando para instalar cun só clic.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Usuarios chineses, use o seguinte comando no seu lugar e establecerase automaticamente o idioma, a zona horaria, etc.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo habilita IPV6

Activa IPV6 para que SMTP tamén poida enviar correos electrónicos con enderezos IPV6.

editar `/etc/sysctl.conf`

Modifica ou engade as seguintes liñas

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Continúa co [tutorial de contabo: Engadir conectividade IPv6 ao teu servidor](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Edita `/etc/netplan/01-netcfg.yaml` , engade algunhas liñas como se mostra na figura seguinte (o ficheiro de configuración predeterminado de Contabo VPS xa ten estas liñas, só tes que descomentalas).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Despois `netplan apply` para facer efectiva a configuración modificada.

Despois de que a configuración teña éxito, pode usar `curl 6.ipw.cn` para ver o enderezo ipv6 da súa rede externa.

## Clonar as operacións do repositorio de configuración

```
git clone https://github.com/wactax/ops.soft.git
```

## Xera un certificado SSL gratuíto para o teu nome de dominio

O envío de correo require un certificado SSL para o cifrado e a sinatura.

Usamos [acme.sh](https://github.com/acmesh-official/acme.sh) para xerar certificados.

acme.sh é unha ferramenta de sinatura automática de certificados de código aberto,

Introduza o almacén de configuración ops.soft, execute `./ssl.sh` e crearase un cartafol `conf` **no directorio superior** .

Busca o teu provedor de DNS en [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , edita `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

A continuación, executa `./ssl.sh 123.com` para xerar certificados `123.com` e `*.123.com` para o teu nome de dominio.

A primeira execución instalará automaticamente [acme.sh](https://github.com/acmesh-official/acme.sh) e engadirá unha tarefa programada para a renovación automática. Podes ver `crontab -l` , hai unha liña como a seguinte.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

O camiño para o certificado xerado é algo así como `/mnt/www/.acme.sh/123.com_ecc。`

A renovación do certificado chamará ao script `conf/reload/123.com.sh` , edita este script, podes engadir comandos como `nginx -s reload` para actualizar a caché de certificados das aplicacións relacionadas.

## Construír un servidor SMTP con chasquid

[chasquid](https://github.com/albertito/chasquid) é un servidor SMTP de código aberto escrito en linguaxe Go.

Como substituto dos antigos programas do servidor de correo, como Postfix e Sendmail, chasquid é máis sinxelo e fácil de usar, e tamén é máis fácil para o desenvolvemento secundario.

Executar `./chasquid/init.sh 123.com` instalarase automaticamente cun só clic (substitúe 123.com polo teu nome de dominio de envío).

## Configurar a sinatura de correo electrónico DKIM

DKIM úsase para enviar sinaturas de correo electrónico para evitar que as cartas sexan tratadas como spam.

Despois de que o comando se execute correctamente, pediráselle que configure o rexistro DKIM (como se mostra a continuación).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Só ten que engadir un rexistro TXT ao seu DNS (como se mostra a continuación).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Consulta o estado e os rexistros do servizo

 `systemctl status chasquid` Ver o estado do servizo.

O estado de funcionamento normal é o que se mostra na seguinte figura

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` ou `journalctl -xeu chasquid` poden ver o rexistro de erros.

## Configuración inversa do nome de dominio

O nome de dominio inverso é para permitir que o enderezo IP se resolva co nome de dominio correspondente.

Establecer un nome de dominio inverso pode evitar que os correos electrónicos sexan identificados como spam.

Cando se reciba o correo, o servidor de recepción realizará unha análise inversa do nome de dominio no enderezo IP do servidor de envío para confirmar se o servidor de envío ten un nome de dominio inverso válido.

Se o servidor de envío non ten un nome de dominio inverso ou se o nome de dominio inverso non coincide co enderezo IP do servidor de envío, o servidor de recepción pode recoñecer o correo electrónico como spam ou rexeitalo.

Visita [https://my.contabo.com/rdns](https://my.contabo.com/rdns) e configura como se mostra a continuación

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Despois de configurar o nome de dominio inverso, recorda configurar a resolución de avance do nome de dominio ipv4 e ipv6 ao servidor.

## Edite o nome de host de chasquid.conf

Modifique `conf/chasquid/chasquid.conf` co valor do nome de dominio inverso.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

A continuación, execute `systemctl restart chasquid` para reiniciar o servizo.

## Copia de seguranza da configuración no repositorio git

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Por exemplo, fago unha copia de seguridade do cartafol conf no meu propio proceso github do seguinte xeito

Crea primeiro un almacén privado

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Introduza o directorio conf e envíeo ao almacén

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Engadir remitente

correr

```
chasquid-util user-add i@wac.tax
```

Pode engadir un remitente

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Verifique que o contrasinal está configurado correctamente

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Despois de engadir o usuario, actualizarase `chasquid/domains/wac.tax/users` , recorda envialo ao almacén.

## DNS engadir rexistro SPF

SPF (Sender Policy Framework) é unha tecnoloxía de verificación de correo electrónico utilizada para evitar fraudes por correo electrónico.

Verifica a identidade dun remitente de correo comprobando que o enderezo IP do remitente coincide cos rexistros DNS do nome de dominio que afirma ser, evitando que os defraudadores envíen correos electrónicos falsos.

Engadir rexistros SPF pode evitar que os correos electrónicos sexan identificados como spam na medida do posible.

Se o teu servidor de nomes de dominio non admite o tipo SPF, só tes que engadir un rexistro de tipo TXT.

Por exemplo, o SPF de `wac.tax` é o seguinte

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF para `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Teña en conta que `include:_spf.google.com` , porque máis tarde configurarei `i@wac.tax` como enderezo de envío na caixa de correo de Google.

## Configuración DNS DMARC

DMARC é a abreviatura de (Domain-based Message Authentication, Reporting & Conformance).

Utilízase para capturar rebotes SPF (quizais causados ​​por erros de configuración ou que outra persoa pretende ser ti para enviar spam).

Engadir rexistro TXT `_dmarc` ,

O contido é o seguinte

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

O significado de cada parámetro é o seguinte

### p (política)

Indica como tratar os correos electrónicos que fallan na verificación SPF (Sender Policy Framework) ou DKIM (DomainKeys Identified Mail). O parámetro p pódese configurar nun dos tres valores:

* none: non se realiza ningunha acción, só se envía o resultado da verificación ao remitente a través do mecanismo de notificación por correo electrónico.
* Corentena: pon o correo que non pasou a verificación no cartafol de spam, pero non o rexeitará directamente.
* rexeitar: rexeita directamente os correos electrónicos que fallan na verificación.

### fo (Opcións de fallo)

Especifica a cantidade de información que devolve o mecanismo de informes. Pódese establecer cun dos seguintes valores:

* 0: informe dos resultados da validación de todas as mensaxes
* 1: Informa só das mensaxes que non se verifican
* d: Informa só de fallos de verificación de nomes de dominio
* s: informe só de fallos de verificación SPF
* l: informe só de fallos de verificación de DKIM

### rua & ruf

* rua (URI de informes para informes agregados): enderezo de correo electrónico para recibir informes agregados
* ruf (URI de informes para informes forenses): enderezo de correo electrónico para recibir informes detallados

## Engade rexistros MX para reenviar correos electrónicos a Google Mail

Como non puiden atopar unha caixa de correo corporativa gratuíta que admita enderezos universais (Catch-All, pode recibir calquera correo electrónico enviado a este nome de dominio, sen restricións de prefixos), usei chasquid para reenviar todos os correos electrónicos á miña caixa de correo de Gmail.

**Se tes a túa propia caixa de correo comercial de pago, non modifiques o MX e omita este paso.**

Editar `conf/chasquid/domains/wac.tax/aliases` , configurar a caixa de correo de reenvío

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` indica todos os correos electrónicos, `i` é o prefixo do enderezo de correo electrónico do usuario remitente creado anteriormente. Para reenviar correo, cada usuario debe engadir unha liña.

A continuación, engade o rexistro MX (apunto directamente ao enderezo do nome de dominio inverso aquí, como se mostra na primeira liña da figura a continuación).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Despois de completar a configuración, podes usar outros enderezos de correo electrónico para enviar correos electrónicos a `i@wac.tax` e `any123@wac.tax` para ver se podes recibir correos electrónicos en Gmail.

Se non, verifique o rexistro de chasquid ( `grep chasquid /var/log/syslog` ).

## Envía un correo electrónico a i@wac.tax con Google Mail

Despois de que Google Mail recibise o correo, naturalmente esperaba responder con `i@wac.tax` en lugar de i.wac.tax@gmail.com.

Visita [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) e fai clic en "Engadir outro enderezo de correo electrónico".

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

A continuación, introduza o código de verificación recibido polo correo electrónico ao que se enviou.

Finalmente, pódese establecer como enderezo predeterminado do remitente (xunto coa opción de responder co mesmo enderezo).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Deste xeito, completamos o establecemento do servidor de correo SMTP e ao mesmo tempo utilizamos Google Mail para enviar e recibir correos electrónicos.

## Envía un correo electrónico de proba para comprobar se a configuración é correcta

Introduce `ops/chasquid`

Executar `direnv allow` instalar dependencias (direnv instalouse no proceso anterior de inicialización cunha tecla e engadiuse un gancho ao shell)

despois corre

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

O significado dos parámetros é o seguinte

* usuario: nome de usuario SMTP
* pass: contrasinal SMTP
* a: destinatario

Podes enviar un correo electrónico de proba.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Recoméndase usar Gmail para recibir correos electrónicos de proba para comprobar se as configuracións son correctas.

### Cifrado estándar TLS

Como se mostra na seguinte figura, existe este pequeno bloqueo, o que significa que o certificado SSL se activou correctamente.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

A continuación, fai clic en "Mostrar correo electrónico orixinal"

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Como se mostra na seguinte figura, a páxina de correo orixinal de Gmail mostra DKIM, o que significa que a configuración de DKIM foi correcta.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Comprobe o Recibido na cabeceira do correo electrónico orixinal e verá que o enderezo do remitente é IPV6, o que significa que IPV6 tamén está configurado correctamente.
