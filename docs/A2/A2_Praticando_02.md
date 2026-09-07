---
puppeteer:
  format: "A4"
  printBackground: true
  margin:
    top: "20mm"
    bottom: "20mm"
    left: "25mm"
    right: "25mm"
---

# A2 - Praticando

## Sistema de Agendamento de Consultas

### Grupo 24

- Caio Henrique Gomes Paulo
- David Pessoa
- Fabio Capalbo Payao Rodrigues
- Jean Carlos Monteiro Santos
- Jessica Isis Medeiros Oliveira

---

## 1. Modelo de Domínio

O modelo de domínio apresenta os principais conceitos envolvidos no Sistema de Agendamento de Consultas e os relacionamentos existentes entre eles.

```plantuml
@startuml

scale 1.2

title Modelo de Domínio - Sistema de Agendamento de Consultas

skinparam classAttributeIconSize 0

class Paciente {
    idPaciente
    nome
    email
    senha
}

class "Médico" as Medico {
    idMedico
    nome
    especialidade
    email
    senha
}

class Agenda {
    idAgenda
    duracaoConsulta
    valorConsulta
}

class "Horário de Atendimento" as HorarioAtendimento {
    data
    horaInicio
    horaFim
}

class Consulta {
    idConsulta
    data
    hora
    duracao
    valor
    status
}

class Lembrete {
    idLembrete
    dataEnvio
    status
}

Paciente "1" -- "0..*" Consulta : agenda
Medico "1" -- "0..*" Consulta : atende
Medico "1" -- "1" Agenda : possui
Agenda "1" *-- "0..*" HorarioAtendimento : contém
Consulta "1" -- "1" HorarioAtendimento : ocupa
Consulta "1" -- "0..1" Lembrete : gera

@enduml
```

---

## 2. Diagramas de Sequência de Sistema

Os diagramas de sequência de sistema representam as interações entre os atores e o Sistema de Agendamento de Consultas durante a execução dos principais casos de uso definidos na fase de concepção.

<div style="page-break-before: always;"></div>

### 2.1 UC02 - Agendar consulta

```plantuml
@startuml

hide footbox
scale 1.2
skinparam maxMessageSize 220

title UC02 - Agendar Consulta

actor Paciente
participant "Sistema de Agendamento\nde Consultas" as Sistema

Paciente -> Sistema : selecionar médico desejado

alt médico possui horários livres

    Sistema --> Paciente : apresentar agenda e horários livres

    Paciente -> Sistema : selecionar horário livre

    Sistema --> Paciente : apresentar resumo da consulta\n(data, horário, duração e valor)

    Paciente -> Sistema : confirmar agendamento

    alt horário permanece disponível

        Sistema --> Paciente : apresentar confirmação do agendamento

    else horário foi reservado por outro paciente

        Sistema --> Paciente : informar indisponibilidade do horário
        Sistema --> Paciente : apresentar agenda atualizada

    else paciente já possui consulta com o mesmo\nmédico na mesma data

        Sistema --> Paciente : informar que já existe consulta agendada
        Sistema --> Paciente : apresentar consulta existente

    end

else médico não possui horários livres

    Sistema --> Paciente : informar ausência de horários
    Sistema --> Paciente : permitir consulta de outras datas

end

@enduml
```

<div style="page-break-before: always;"></div>

### 2.2 UC03 - Cancelar consulta

O caso de uso UC03 é representado em dois diagramas, correspondentes aos dois atores que podem originar o cancelamento.

#### 2.2.1 Cancelamento solicitado pelo paciente

```plantuml
@startuml

hide footbox
scale 1.2
skinparam maxMessageSize 220

title UC03 - Cancelar Consulta (solicitado pelo Paciente)

actor Paciente
participant "Sistema de Agendamento\nde Consultas" as Sistema

Paciente -> Sistema : acessar consultas agendadas

Sistema --> Paciente : apresentar consultas do paciente

Paciente -> Sistema : selecionar consulta

Paciente -> Sistema : solicitar cancelamento

alt consulta está dentro do prazo mínimo\ndefinido para cancelamento

    Sistema --> Paciente : informar restrição de cancelamento
    Sistema --> Paciente : informar que a consulta permanece agendada

else cancelamento permitido

    Sistema --> Paciente : solicitar confirmação

    alt paciente confirma

        Paciente -> Sistema : confirmar cancelamento

        Sistema --> Paciente : informar cancelamento realizado

    else paciente desiste

        Paciente -> Sistema : desistir da operação

        Sistema --> Paciente : informar que a consulta permanece agendada

    end

end

@enduml
```

<div style="page-break-before: always;"></div>

#### 2.2.2 Cancelamento solicitado pelo médico

```plantuml
@startuml

hide footbox
scale 1.2
skinparam maxMessageSize 220

title UC03 - Cancelar Consulta (solicitado pelo Médico)

actor "Médico" as Medico
participant "Sistema de Agendamento\nde Consultas" as Sistema

Medico -> Sistema : acessar consultas da agenda

Sistema --> Medico : apresentar consultas agendadas

Medico -> Sistema : selecionar consulta

Medico -> Sistema : solicitar cancelamento

Sistema --> Medico : solicitar confirmação

alt médico confirma

    Medico -> Sistema : confirmar cancelamento

    Sistema --> Medico : informar cancelamento realizado

else médico desiste

    Medico -> Sistema : desistir da operação

    Sistema --> Medico : informar que a consulta permanece agendada

end

@enduml
```

<div style="page-break-before: always;"></div>

### 2.3 UC06 - Manter agenda de atendimento

```plantuml
@startuml

hide footbox
scale 1.2
skinparam maxMessageSize 220

title UC06 - Manter Agenda de Atendimento

actor "Médico" as Medico
participant "Sistema de Agendamento\nde Consultas" as Sistema

Medico -> Sistema : acessar agenda de atendimento

Sistema --> Medico : apresentar períodos cadastrados

Medico -> Sistema : incluir, alterar ou remover período\n(dia, horário, duração e valor)

alt período se sobrepõe a outro já cadastrado

    Sistema --> Medico : informar conflito de horários
    Sistema --> Medico : informar que a alteração não foi realizada

else alteração afeta consultas já agendadas

    Sistema --> Medico : apresentar consultas afetadas
    Sistema --> Medico : informar que as consultas devem ser\ntratadas antes da alteração

else dados válidos

    Sistema --> Medico : informar que a agenda foi atualizada
    Sistema --> Medico : apresentar agenda atualizada

end

@enduml
```

<div style="page-break-before: always;"></div>

### 2.4 UC07 - Consultar pacientes do dia

```plantuml
@startuml

hide footbox
scale 1.2
skinparam maxMessageSize 220

title UC07 - Consultar Pacientes do Dia

actor "Médico" as Medico
participant "Sistema de Agendamento\nde Consultas" as Sistema

Medico -> Sistema : acessar painel do dia

alt existem consultas agendadas

    Sistema --> Medico : apresentar pacientes em ordem de horário
    Sistema --> Medico : apresentar horários livres

    alt médico informou valor dos atendimentos

        Sistema --> Medico : apresentar previsão de horas\nde atendimento
        Sistema --> Medico : apresentar valor previsto a receber

    else médico não informou valor dos atendimentos

        Sistema --> Medico : apresentar previsão de horas\nde atendimento

    end

else não existem consultas agendadas

    Sistema --> Medico : informar ausência de consultas
    Sistema --> Medico : apresentar agenda inteiramente livre

end

Medico -> Sistema : selecionar outra data

Sistema --> Medico : apresentar informações da data selecionada

@enduml
```
