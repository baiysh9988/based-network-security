# based-network-security
<p align="center">
  <img src="https://github.com/baiysh9988/based-network-security/blob/main/basic%20network%20security.png" alt="Secure Network Topology" width="800">
</p>

<hr>

<h2>🏗️ 1. Общая структура</h2>
<p>
Сеть разделена на несколько уровней:
</p>
<ul>
  <li><strong>Access Layer</strong> — рабочие станции, управляемый коммутатор.</li>
  <li><strong>Perimeter Layer</strong> — маршрутизатор с NAT и ACL-фильтрацией.</li>
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
  <li><strong>Brute-force защита:</strong> ограничение попыток входа и блокировка.</li>
  <li><strong>Пользователи:</strong> enable secret(baiysh), username(baiysh) с паролем(baiysh) в хешированном виде и с уровнем доступа privilege level 0.</li>
  <li><strong>Управление доступом (ACL на SSH):</strong>Настроен ACL на доступ к оборудования по ssh, то есть доступ к оборудованию можно получить только с адреса 192.168.2.2(admin).</li>
</ul>

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
  <li>Выступает в качестве шлюза для всех сегментов</li>
  <li><strong>ZONE-BASED FIREWALL</strong> не настроен, из-за ограничений cisco pocket tracer</li>
  <li>Есть ACL запрещающий обращение к виртуальным интерфейсам маршрутизатора, кроме 192.168.5.1</li>
  <li>SSH-only доступ, ACL ограничивает вход. То есть можно с 192.168.2.2(admin) на 192.168.5.1(MGMT и шлюз)</li>
  <li><strong>AAA new-model, enable secret и пользователи</strong> Всё также, как и с коммутатором.</li>
  <li>Защита от brute-force и широковещательных штормов.</li>
  <li>Сегменты MGMT и SERVERS исключены из NAT, также стоит ACL исключающий эти сегменты из сети Интернет</li>
</ul>

<hr>

<h2>📋 4. Модель RBAC (Role-Based Access Control)</h2>

<table border="1" cellspacing="0" cellpadding="6">
  <tr style="background-color:#f2f2f2;">
    <th>VLAN</th><th>INTERNET</th><th>SERVER</th><th>MGMT</th>
  </tr>
  <tr><td>Admins</td><td>✅</td><td>❌</td><td>✅</td></tr>
  <tr><td>Managers</td><td>✅</td><td>✅</td><td>❌</td></tr>
  <tr><td>Salers</td><td>✅</td><td>❌</td><td>❌</td></tr>

</table>

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
