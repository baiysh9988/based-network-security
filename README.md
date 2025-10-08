# based-network-security
<h1 align="center">🛡️ Secure Network Topology v2</h1>

<p align="center">
  <img src="64cc71f1-1308-4c38-9618-c5224b37b726.png" alt="Secure Network Topology" width="800">
</p>

<hr>

<h2>🏗️ 1. Общая структура</h2>
<p>
Сеть разделена на несколько уровней:
</p>
<ul>
  <li><strong>Access Layer</strong> — рабочие станции, управляемый коммутатор.</li>
  <li><strong>Perimeter Layer</strong> — маршрутизатор с NAT и ACL-фильтрацией.</li>
  <li><strong>Provider Layer</strong> — подключение к внешнему серверу.</li>
</ul>

<p>
Используется сегментация по VLAN и модель <strong>RBAC (Role-Based Access Control)</strong> для разграничения доступа.
</p>

<hr>

<h2>🧩 2. Уровень доступа (Access Layer)</h2>

<p><strong>Устройство:</strong> Cisco 2960 Switch</p>

<h3>🔹 Функции безопасности</h3>

<ul>
  <li><strong>Port Security:</strong> ограничение 1 MAC-адреса на порт, sticky, режим restrict.</li>
  <li><strong>Storm Control:</strong> предотвращение широковещательных штормов.</li>
  <li><strong>AAA new-model:</strong> централизованная аутентификация пользователей.</li>
  <li><strong>SSH-only access:</strong> Telnet отключён, доступ только по SSH.</li>
  <li><strong>Brute-force защита:</strong> ограничение попыток входа.</li>
  <li><strong>Пароли:</strong> enable secret, username с уровнем доступа privilege level 15.</li>
</ul>

<h3>🔹 Управление доступом (ACL на SSH)</h3>
<pre>
ip access-list standard SSH_ACCESS
 permit 192.168.2.2
!
line vty 0 4
 transport input ssh
 access-class SSH_ACCESS in
 login local
</pre>

<h3>🔹 VLAN-сегментация</h3>
<table border="1" cellspacing="0" cellpadding="6">
  <tr style="background-color:#f2f2f2;">
    <th>VLAN</th><th>Сегмент</th><th>Подсеть</th><th>Назначение</th>
  </tr>
  <tr><td>2</td><td>ADMIN</td><td>192.168.2.0/24</td><td>Администраторы</td></tr>
  <tr><td>3</td><td>MANAGERS</td><td>192.168.3.0/24</td><td>Менеджеры</td></tr>
  <tr><td>4</td><td>SALERS</td><td>192.168.4.0/24</td><td>Отдел продаж</td></tr>
  <tr><td>5</td><td>MGMT</td><td>192.168.5.0/24</td><td>Сегмент управления оборудованием</td></tr>
  <tr><td>6</td><td>SERVERS</td><td>192.168.6.0/24</td><td>Локальные серверы</td></tr>
</table>

<hr>

<h2>🧱 3. Уровень периметра (Perimeter Router)</h2>

<p><strong>Устройство:</strong> Cisco ISR 4331</p>

<h3>🔹 Функции безопасности</h3>
<ul>
  <li>Разрешено управление только с IP <code>192.168.5.2</code> (MGMT VLAN).</li>
  <li>SSH-only доступ, ACL ограничивает вход.</li>
  <li><strong>AAA new-model</strong> с локальной аутентификацией.</li>
  <li><strong>Enable secret</strong> и индивидуальные пользователи с уровнями привилегий.</li>
  <li>Защита от brute-force и широковещательных штормов.</li>
</ul>

<h3>🔹 Пример конфигурации доступа</h3>
<pre>
ip access-list standard MGMT_ACCESS
 permit 192.168.5.2
!
line vty 0 4
 transport input ssh
 access-class MGMT_ACCESS in
 login local
</pre>

<h3>🔹 Маршрутизация и NAT</h3>
<pre>
interface Gi0/0.2
 encapsulation dot1Q 2
 ip address 192.168.2.1 255.255.255.0
 ip nat inside
!
interface Gi0/0.3
 encapsulation dot1Q 3
 ip address 192.168.3.1 255.255.255.0
 ip nat inside
!
interface Gi0/0.4
 encapsulation dot1Q 4
 ip address 192.168.4.1 255.255.255.0
 ip nat inside
!
interface Gi0/0.5
 encapsulation dot1Q 5
 ip address 192.168.5.1 255.255.255.0
!
interface Gi0/0.6
 encapsulation dot1Q 6
 ip address 192.168.6.1 255.255.255.0
!
interface Gi0/1
 ip address 10.1.1.1 255.255.255.252
 ip nat outside
!
ip nat inside source list 1 interface Gi0/1 overload
access-list 1 permit 192.168.2.0 0.0.3.255
!
ip route 0.0.0.0 0.0.0.0 10.1.1.2
</pre>

<p><em>Сегменты MGMT и SERVERS исключены из NAT и не имеют выхода в Интернет.</em></p>

<hr>

<h2>📋 4. Модель RBAC (Role-Based Access Control)</h2>

<table border="1" cellspacing="0" cellpadding="6">
  <tr style="background-color:#f2f2f2;">
    <th>Роль</th><th>VLAN</th><th>Доступ к сегментам</th><th>Выход в Интернет</th>
  </tr>
  <tr><td>Admin</td><td>2</td><td>MGMT (VLAN5)</td><td>✅</td></tr>
  <tr><td>Managers</td><td>3</td><td>SERVERS (VLAN6)</td><td>✅</td></tr>
  <tr><td>Salers</td><td>4</td><td>Только Интернет</td><td>✅</td></tr>
  <tr><td>MGMT</td><td>5</td><td>Только внутренний доступ</td><td>❌</td></tr>
  <tr><td>Servers</td><td>6</td><td>Только от Managers</td><td>❌</td></tr>
</table>

<hr>

<h2>☁️ 5. Провайдер и сервер</h2>

<p><strong>Provider Router:</strong> 10.1.1.2/30 → 10.1.2.1/30</p>
<p><strong>Server:</strong> 10.1.2.2/30 (внешний сервис)</p>

<p>
Провайдер выполняет роль шлюза к Интернету. 
На стороне периметра — NAT (PAT), скрывающий внутренние сети.
</p>

<hr>

<h2>🧠 6. Безопасность и принципы защиты</h2>

<ul>
  <li>Многоуровневая сегментация VLAN.</li>
  <li>Ограничение управления оборудованием (только SSH, только из MGMT).</li>
  <li>Ролевая модель доступа (RBAC).</li>
  <li>Изоляция служебных сегментов от Интернета.</li>
  <li>NAT с минимальным правилом (только рабочие подсети).</li>
  <li>Политика паролей и привилегий на всех устройствах.</li>
  <li>Базовая защита L2 (Port Security, Storm Control).</li>
  <li>Базовая защита L3 (ACL, SSH, AAA, Anti-BruteForce).</li>
</ul>

<hr>

<p align="center"><em>Автор проекта: <strong>MV Alvert</strong></em></p>
