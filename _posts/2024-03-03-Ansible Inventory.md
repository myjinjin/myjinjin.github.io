---
layout: post
title: "[Ansible] 03_Ansible Inventory"
date: 2024-03-02 15:00:00
categories: Ansible
---

---

> ref: https://www.udemy.com/course/learn-ansible/ 

---

# Ansible Inventory

## inventory

Ansible에서 inventory 구성에 대해 살펴보겠다. Ansible은 인프라의 하나 또는 여러 시스템에서 동시에 작업할 수 있다. 여러 서버에서 작업하려면 Ansible에서 해당 서버들에 대한 연결을 설정해야 한다. 이 작업은 <span class="important">Linux용 SSH</span> 및 <span class="important">Windows용 PowerShell Remoting</span>을 사용하여 수행된다. 이것이 바로 Ansible을 <span class="important">Agentless</span>하게 만들어준다!

> 💡 Agentless? 
> 
> Agentless하다는 것은, Ansible을 사용하기 위해 대상 컴퓨터에 추가 소프트웨어를 설치할 필요가 없음을 의미한다.

간단한 SSH 연결만으로도 Ansible의 요구 사항을 충족할 수 있다. 대부분의 다른 오케스트레이션 도구의 주요 단점 중 하나는 모든 종류의 자동화를 호출하기 전에 대상 시스템에서 에이전트를 구성해야 한다는 것이다.

이러한 대상 시스템에 대한 정보는 inventory 파일에 저장된다. 새 inventory 파일을 만들지 않는 경우, Ansible은 `/etc/ansible/hosts` 위치에 있는 기본 inventory 파일을 사용한다.

inventory 파일 샘플을 살펴보겠다. inventory 파일은 INI와 같은 형식이다.

```ini
server1.company.com
server2.company.com

[mail]
server3.company.com
server4.company.com

[db]
server5.company.com
server6.company.com

[web]
server7.company.com
server8.company.com
```

단순히 여러 대의 서버를 차례대로 나열하는 것이다. 대괄호 안의 그룹 이름 아래에 다음과 같이 정의하여 여러 서버를 함께 그룹화하고 아래 줄에 해당 그룹에 속하는 서버 목록을 정의할 수도 있다. 하나의 inventory 파일에 여러 그룹을 정의할 수 있다.

## More on inventory files

inventory 파일에 대해 자세히 살펴보겠다.

예를 들어, 1부터 4까지 이름이 지정된 서버 목록이 있다. 하지만 웹 서버 또는 데이터베이스 서버와 같은 별칭을 사용하여 Ansible에서 이러한 서버를 참조하고 싶다.

각 서버의 별칭을 줄의 시작 부분에 추가하고 해당 서버의 주소를 `ansible_host` 파라미터에 할당하면 이 작업을 수행할 수 있다.

```ini
web     ansible_host=server1.company.com
db      ansible_host=server2.company.com
mail    ansible_host=server3.company.com
web2    ansible_host=server4.company.com
```

`ansible_host`는 서버의 FQDN 또는 IP 주소를 지정하는 데 사용되는 inventory 파라미터이다. 다른 inventory 파라미터들도 있다. 그중 일부는 다음과 같다.

- `ansible_connection`
    - ssh/winrm/localhost
    - Ansible이 대상 서버에 연결하는 방법 정의
- `ansible_port`
    - 22/5986
    - 연결할 포트 정의
- `ansible_user`
    - root/administrator
    - 원격 연결에 사용되는 사용자 정의
- `ansible_ssh_pass`
    - Password
    - Linux용 SSH 암호 정의
    - 이와 같은 일반 텍스트 형식으로 암호를 저장하는 것은 이상적이지 않다. 가장 좋은 방법은 서버 간에 SSH 키 기반의 암호 없는 인증을 설정하는 것이며, 프로덕션 환경이나 회사 환경에서는 반드시 그렇게 해야 한다!

```ini
web     ansible_host=server1.company.com    ansible_connection=ssh      ansible_user=root
db      ansible_host=server2.company.com    ansible_connection=winrm    ansible_user=admin
mail    ansible_host=server3.company.com    ansible_connection=ssh      ansible_ssh_pass=P@SS!
web2    ansible_host=server4.company.com    ansible_connection=winrm

localhost ansible_connection=localhost
```

---

# Inventory Formats

## Why Do We Need Different Inventory Formats?

두 회사의 예시를 들어보겠다.

- Start-up
- Multinational Corporation

첫 번째는 웹 호스팅 및 데이터베이스 관리와 같은 기본적인 기능을 처리하는 몇 가지 서비스를 제공하는 소규모 스타트업이다. 두 번째는 전 세계에 수백 대의 서버를 두고 전자상거래, 고객 지원, 데이터 분석 등 다양한 기능을 처리하는 다국적 기업이다.

소규모 스타트업의 경우 간단한 INI 형식의 인벤토리로 충분할 것이다. 이는 몇 개의 부서만 있는 기본적인 조직도와 같다. 하지만 다국적 기업의 경우, 보다 상세하고 구조화된 인벤토리가 필요하다. 바로 이 점이 YAML 형식이 필요한 이유이다. 이는 다양한 부서, 하위 부서, 팀으로 구성된 복잡한 조직도와 같다. 

또한 다양한 인벤토리 형식은 <span class="important">유연성(Flexibility)</span>을 제공하여 웹 서버, 데이터베이스 서버, 애플리케이션 서버 등의 역할, 미국, 유럽, 아시아 서버 등의 <span class="important">지리적 위치(Geographical Location)</span> 또는 조직에 적합한 기타 기준에 따라 서버를 그룹화할 수 있다.

## Different Inventory Format Types

Ansible은 두 가지 주요 유형의 인벤토리 형식을 지원한다. 즉, `INI`와 `YAML`을 지원한다.

### `INI`

- 가장 간단하고 직관적이다.
- 소규모 스타트업의 기본 조직도와 비슷하다.
- 예시
    ```ini
    [webservers]
    web1.example.com
    web2.example.com

    [dbservers]
    db1.example.com
    db2.example.com
    ```

### `YAML`

- 더 체계적이고 유연하다.
- 다국적 기업의 복잡한 조직도와 비슷하다.
- 예시
    ```yaml
    all:
      children:
        webservers:
          hosts:
            web1.example.com:
            web2.example.com:
        dbservers:
          hosts:
            db1.example.com:
            db2.example.com:
    ```


회사가 운영에 적합한 조직도를 선택하는 것처럼, 프로젝트의 요구와 복잡성에 가장 적합한 인벤토리 형식을 선택해야 한다. `INI` 형식을 선택하든, `YAML` 형식을 선택하든, 핵심은 서버와 부서가 조직 목표를 달성하기 위해 함께 일할 수 있도록 하는 것이다. 

---

# Grouping and Parent-Child Relationship

## Introduction

복잡한 IT 인프라를 갖춘 대기업의 IT 관리자라고 상상해보자. 여러 위치에 분산되어 다양한 용도로 사용되는 수백 대의 서버가 있다. 일부는 웹 서버, 일부는 데이터베이스 서버, 일부는 애플리케이션 서버일 수 있다.

## Why Do We Need Grouping?

이러한 서버를 개별적으로 관리하는 것은 어려운 작업이 될 수 있다. 예를 들어 웹 서버를 업데이트해야 하는 경우 각 웹 서버를 수동으로 지정해야 한다. 이 작업은 <span class="important">시간이 오래 걸릴</span> 뿐만 아니라 <span class="important">오류가 발생하기 쉽다</span>.

복잡한 IT 인프라를 관리할 때 서버의 역할, 위치 또는 우리 환경에 적합한 기타 기준에 따라 서버를 분류할 수 있다면 편리하지 않을까?

예를 들어 웹 서버라는 공통 레이블로 모든 웹 서버를 일괄적으로 식별할 수 있다면, 웹 서버에 대한 업데이트가 필요할 때 각 서버를 수동으로 지정하는 대신 웹 서버 레이블을 대상으로 지정할 수 있다. 그러면 해당 레이블과 관련된 모든 서버에 변경 사항이 적용되어 시간을 절약하고 오류의 위험을 줄일 수 있다.

바로 이 지점에서 Ansible의 그룹화 기능이 작동한다.

Ansible의 그룹화를 사용하면, 이러한 공통 레이블 또는 그룹을 만들어 서버 집합을 한 곳에서 효율적으로 관리하고 운영할 수 있다.

## Why Do We Need Parent-Child Relationships?

웹 서버를 업데이트해야 할 때 웹 서버 그룹을 대상으로 지정하기만 하면 해당 그룹의 모든 서버에 변경 사항을 적용할 수 있다.

하지만 웹 서버가 여러 위치에 분산되어 있고 각 위치의 서버에 위치별 구성이 필요한 경우에는 어떻게 해야 할까?

각 위치에 웹 서버를 위한 별도의 그룹을 만들 수 있지만 그렇게 하면 많은 공통 구성이 중복된다.

이 때 Ansible의 부모-자식 관계가 유용하게 사용된다. 웹 서버라는 상위 그룹과 각 위치에 대한 하위 그룹(예: 미국 웹 서버, 유럽 웹 서버 등)을 만들 수 있다.

상위 그룹 수준에서 공통 구성을 정의하고 하위 그룹 수준에서 위치별 구성을 정의할 수 있다. 이렇게 하면 서버를 보다 효율적으로 관리하고 구성 중복을 방지할 수 있다.

`INI` 형식에서 그룹을 대괄호를 사용하여 정의되며 호스트는 해당 그룹 아래에 나열된다. 부모-자식 관계는 `:children` 접미사를 사용하여 정의한다.

```ini
[webservers:children]
webservers_us
webservers_eu

[webservers_us]
server1_us.com ansible_host=192.168.8.101
server2_us.com ansible_host=192.168.8.102

[webservers_eu]
server1_eu.com ansible_host=10.12.0.101
server2_eu.com ansible_host=10.12.0.102
```

이 예시에서 webservers_us와 webservers_eu는 그룹이고 webservers는 webservers_us와 webservers_eu를 모두 하위 그룹으로 포함하는 상위 그룹이다.

`YAML` 형식에서 그룹은 `hosts` 키워드를 사용하여 정의하고 부모-자식 관계는 `children` 키워드를 사용하여 정의한다. 동일한 예시를 YAML 형식으로 표시하면 아래와 같다.

```yaml
all:
  children:
    webservers:
      children:
        webservers_us:
          hosts:
            server1_us.com:
              ansible_host: 192.168.8.101
            server2_us.com:
              ansible_host: 192.168.8.102
        webservers_eu:
          hosts:
            server1_eu.com:
              ansible_host: 10.12.0.101
            server2_eu.com:
              ansible_host: 10.12.0.102
```
