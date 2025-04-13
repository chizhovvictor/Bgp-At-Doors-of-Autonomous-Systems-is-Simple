# README: VXLAN with BGP EVPN (RFC 7432) Implementation

## Overview

This task extends your knowledge of VXLAN by incorporating **BGP EVPN (RFC 7432)** for automated MAC address learning and dynamic route reflection. In this setup, you will configure VXLAN with an ID of 10 and use BGP EVPN for dynamic, automatic learning of MAC addresses without relying on MPLS.

In this exercise, we simulate a small data center network, where VXLAN-based virtual tunnels are established between VTEPs (VXLAN Tunnel Endpoints). The use of **route reflection (RR)** will simplify the configuration by handling BGP updates between leaf VTEPs, allowing for dynamic relationships among VTEPs.

This README outlines the steps to configure and verify the system, as well as the expected behavior of the network during different stages of setup.

## Prerequisites

Before you begin, make sure you have the following:

1. A network of VTEPs (VXLAN Tunnel Endpoints) connected in a typical leaf-spine architecture.
2. A working VXLAN network, where the VXLAN ID is configured to `10`.
3. Basic knowledge of BGP EVPN and VXLAN.

### Topology

- You will have several **VTEPs** (which are your routers or network devices) participating in the VXLAN network, identified as `wil-1`, `wil-2`, `wil-3`, and `wil-4`.
- These VTEPs will be interconnected, with **VTEP `wil-4`** acting as the **Route Reflector (RR)** for the network.
- The **MAC addresses** of hosts are learned dynamically by the network as the hosts are brought online.
  
### Expected Topology

```
             +--------------------+
             |    wil-4 (RR)      |
             |                    |
             +--------------------+
                   |     |     |
                   |     |     |
          +--------+     |     +--------+
          |              |              |
     +----v----+     +---v----+     +----v----+
     | wil-2   |     | wil-3  |     | wil-1   |
     | (VTEP)  |     | (VTEP) |     | (VTEP)  |
     +---------+     +--------+     +---------+
         |               |               |
   +-----v-----+     +---v----+     +-----v-----+
   | Host 1    |     | Host 2 |     | Host 3    |
   +-----------+     +--------+     +-----------+
```

- **VTEP wil-4** is configured as the **Route Reflector (RR)**.
- **VTEPs wil-1, wil-2, wil-3** act as leaf nodes (hosts and VTEPs).
  
### VXLAN Configuration

- VXLAN ID: **10**
- Bridge: **br0** (created in each VTEP).
- Each VTEP will configure VXLAN with dynamic learning using **BGP EVPN**.

### Workflow

1. **VXLAN Setup**: Set up the VXLAN ID of 10 across all VTEPs.
2. **BGP EVPN**: Use BGP EVPN to learn MAC addresses dynamically and propagate routes for VMs connected to the network.
3. **Dynamic Route Reflection**: Configure Route Reflector (RR) in VTEP `wil-4` to facilitate BGP route reflection and ensure proper route propagation.
4. **MAC Address Learning**: As hosts come online (such as `host_wil-1`, `host_wil-2`, and `host_wil-3`), their MAC addresses will be learned automatically by the VTEPs without requiring manual IP assignments.
5. **Route Generation**: Upon the addition of hosts, the system generates **Type 2 routes** (MAC address routes), propagated through BGP EVPN.

### Verification

1. **Initial State (No Hosts)**:
   - When no hosts are up, the system will see the VNI (VXLAN Network Identifier) 10 and the pre-configured BGP EVPN Type 3 routes. These routes will be used to advertise the VXLAN network to all VTEPs.
   - **Type 2 routes** (host MAC addresses) will not yet be present.

2. **When Host `host_wil-1` is Added**:
   - The VTEP (`wil-2`) will automatically discover the MAC address of `host_wil-1` and create a Type 2 route.
   - The Route Reflector (RR) will propagate the new MAC address route to all VTEPs.
   
3. **When Host `host_wil-3` is Added**:
   - The VTEP (`wil-4`) will automatically discover the MAC address of `host_wil-3` and create another Type 2 route.
   - The Route Reflector (RR) will propagate the new MAC address route to all VTEPs.

4. **Ping Test**:
   - After the hosts are online and their MAC addresses are learned by the network, a simple **ping** test between the hosts will confirm that all machines can communicate.
   - This confirms that VXLAN with BGP EVPN (without MPLS) is functioning correctly.

5. **Expected Output**:
   - You will see the VXLAN ID 10 configured, as well as ICMP packets being exchanged between the hosts.
   - You will also see OSPF packets exchanged between VTEPs to handle network layer routing.

## Configuration Steps

1. **Set up VXLAN on all VTEPs**:
   - Each VTEP should be configured with a bridge (`br0`) and a VXLAN interface (e.g., `vxlan10`).
   - The IP address for each VTEP's `lo` (loopback) interface should be configured with a unique IP.

2. **Configure BGP EVPN**:
   - Set up BGP EVPN with Route Reflector (RR) functionality on `wil-4`.
   - Configure each VTEP with BGP to communicate with `wil-4` (the RR) and exchange MAC address routes.

3. **Dynamic MAC Address Learning**:
   - Ensure that the BGP EVPN configuration includes dynamic MAC address learning and route advertisement.

4. **Bring Hosts Online**:
   - Bring each host online and verify that the MAC addresses are automatically learned by the VTEPs.

5. **Testing**:
   - Perform a ping test to verify that the VTEPs can reach each other and that the MAC address routes are functioning.

## Troubleshooting

- **No MAC Address Learning**: Ensure that BGP EVPN is correctly configured on all VTEPs and that the Route Reflector is properly propagating routes.
- **Ping Fails**: Verify VXLAN tunnel status and ensure that the VTEPs can communicate with each other over the tunnel.
- **Route Propagation**: Ensure the Route Reflector is receiving and reflecting BGP routes correctly.

## Conclusion

In this exercise, you’ve successfully configured a VXLAN network using BGP EVPN for dynamic MAC address learning. You’ve also learned how to use a **Route Reflector** (RR) to simplify the BGP configuration, allowing for automatic MAC address discovery and route advertisement between VTEPs. The system should now be able to dynamically learn and propagate MAC addresses, providing seamless connectivity between hosts in the VXLAN network.



MPLS (Multiprotocol Label Switching)

MPLS (Multiprotocol Label Switching) — это технология маршрутизации сетевого трафика, которая ускоряет его передачу и делает более эффективным управление потоками данных.

Как работает MPLS:

1) Маршруты на основе меток (Labels):
- Вместо традиционной маршрутизации по IP-адресам каждый пакет данных получает специальную метку (label).
- Эта метка присваивается на входе в MPLS-сеть и используется для быстрого перенаправления пакета по предопределённому маршруту.

2) Label-Switched Path (LSP):
- Перед началом передачи данных создаётся "транспортный туннель" — последовательность маршрутизаторов, которые обрабатывают пакеты только на основе меток.
- Пакет "перескакивает" через маршрутизаторы, которые не анализируют заголовок IP, а просто перенаправляют данные по метке.

Где используется:
- Корпоративные сети: Обеспечивает высокую производительность и гарантии доставки данных между филиалами.
- Телекоммуникационные операторы: MPLS упрощает управление сложными сетями и позволяет предоставлять услуги VPN и QoS.
- ЦОД и провайдеры облачных услуг: Для оптимизации маршрутов и изоляции клиентов.

Почему некоторые уходят от MPLS?
- Высокая стоимость оборудования и обслуживания.
- Переход к более простым и дешевым технологиям, например, VXLAN и SD-WAN.

Объяснение простыми словами:
Представь себе автомагистраль с множеством полос и пунктов назначения.

Обычная маршрутизация:
Каждый автомобиль (пакет данных) останавливается на каждом перекрестке (маршрутизаторе), чтобы узнать, куда ему ехать дальше. Это медленно.

MPLS:
Вместо того чтобы каждый раз проверять маршрут, машине сразу дают наклейку с номером маршрута (лейбл). Все перекрестки просто смотрят на эту наклейку и мгновенно направляют машину по нужной полосе.

Пример:
Ты отправляешь данные из Москвы в Париж. Вместо того чтобы на каждом узле проверять маршрут, данные получают метку "Париж" и быстро проходят через все маршрутизаторы.

route reflection (=RR)
Route Reflection (RR) — это механизм оптимизации маршрутизации в сетях, работающих по протоколу BGP (Border Gateway Protocol). Он позволяет избежать полного "мэшинга" BGP-соединений между маршрутизаторами внутри автономной системы (AS).

По умолчанию BGP требует, чтобы все маршрутизаторы внутри AS создавали полные одноранговые (full-mesh) подключения друг с другом, чтобы обмениваться маршрутной информацией. В крупной сети это приводит к огромному количеству соединений и сложному управлению.

Route Reflection решает эту проблему, позволяя одному маршрутизатору выступать в роли рефлектора маршрутов (Route Reflector).

Как работает Route Reflection:
1) Route Reflector (RR) — маршрутизатор, который собирает информацию о маршрутах от своих клиентов и "отражает" её другим клиентам.
2) Вместо того чтобы все маршрутизаторы обменивались маршрутами друг с другом напрямую, они общаются только с RR.
3) RR обрабатывает и пересылает маршруты между клиентами и другими RR или внешними маршрутизаторами.

Ообъяснение простыми словами:
Представь, что у тебя есть группа друзей (маршрутизаторы), и каждый друг должен знать обо всех маршрутах.
Без Route Reflector:
Каждый друг звонит каждому, чтобы сообщить о новых маршрутах. Это хаос и занимает много времени.

С Route Reflector:
Один друг становится "главным информатором" и собирает все маршруты, а потом делится ими с остальными. Это упрощает общение.

Пример:
В компании есть менеджер, который собирает отчеты от всех сотрудников и потом рассылает информацию. Это RR.

Leaf (VTEP) 
— это термин из сетевых архитектур Clos и EVPN/VXLAN, активно применяемых в современных data center сетях.

В архитектуре Clos (Spine-Leaf):

- Leaf (лист) — это конечные маршрутизаторы или коммутаторы, которые подключаются напрямую к серверам, хранилищам данных и другим сетевым устройствам.
- Все Leaf устройства также подключены к верхнеуровневым коммутаторам Spine (хребет).

VTEP (VXLAN Tunnel Endpoint)
VTEP — это функциональность, которая позволяет маршрутизатору или коммутатору терминировать VXLAN туннели.

VXLAN (Virtual Extensible LAN, RFC 7348) — это технология для расширения сетей L2 через L3-инфраструктуру.
VTEP выполняет инкапсуляцию и декапсуляцию трафика VXLAN между узлами.

Leaf с функцией VTEP:
Каждый коммутатор Leaf может выступать в роли VTEP, создавая виртуальные туннели для передачи данных между различными сегментами сети поверх IP-сети.

Как работают Leaf (VTEP) в сети:

Сервер подключается к Leaf-коммутатору.
Если серверы находятся в разных сегментах сети, трафик инкапсулируется в VXLAN-пакеты.
VXLAN-пакеты передаются через Spine к другому Leaf, который также выполняет роль VTEP.
VTEP второго Leaf декапсулирует пакеты и передает их конечному устройству.

Преимущества архитектуры Leaf (VTEP):

Высокая масштабируемость: Возможность горизонтального увеличения количества Leaf-коммутаторов.
Производительность: Минимизация задержек благодаря плоской структуре сети.
Поддержка мульти-арендности: Изоляция трафика благодаря VXLAN.
Гибкость маршрутизации: Легкость интеграции с EVPN и динамическими протоколами маршрутизации.

Когда применяется:

В облачных и современных центрах обработки данных (Data Centers).
Для реализации виртуальных сетей поверх IP-инфраструктуры.
В сценариях, требующих мульти-арендной изоляции и масштабирования сети.

Leaf/VTEP — основа для построения отказоустойчивых и гибких сетей в современных ЦОДах.

Простыми словами:
Представь себе многоквартирный дом (дата-центр).

Leaf (лист):

Это коммутаторы на каждом этаже, к которым подключены квартиры (серверы).
Spine (хребет):
Это центральные лифты, которые соединяют все этажи.

VTEP:

Представь, что у тебя есть секретный туннель между квартирами на разных этажах. VTEP открывает и закрывает двери этого туннеля, чтобы данные могли пройти.

Пример:
Ты хочешь, чтобы данные между серверами передавались безопасно и быстро через большой офисный комплекс. Leaf/VTEP создают этот "прямой путь" поверх обычной сети.


