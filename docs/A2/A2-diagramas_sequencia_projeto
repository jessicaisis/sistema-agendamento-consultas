# Diagramas de Sequência de Projeto

## Sistema de Agendamento de Consultas

### Grupo 24

Este documento apresenta um rascunho dos diagramas de sequência de projeto dos casos de uso principais do Sistema de Agendamento de Consultas.

> **Premissa deste rascunho:** como o repositório ainda não contém implementação nem diagramas de classes de projeto, os diagramas adotam uma arquitetura web genérica em camadas. Os nomes das classes e operações deverão ser conferidos quando o diagrama de classes de projeto for concluído.

## 1. Organização adotada

Os participantes dos diagramas foram organizados segundo suas responsabilidades:

- **Tela (`boundary`)**: recebe as ações do usuário e apresenta os resultados.
- **Controller (`control`)**: recebe as requisições da interface.
- **Service (`control`)**: executa as regras de negócio do caso de uso.
- **Repository (`repository`)**: consulta e persiste os dados.
- **Serviço auxiliar (`service`)**: executa lembretes e notificações.

Os diagramas cobrem os casos de uso UC02, UC03, UC06 e UC07 detalhados na primeira versão da documentação.

---

## 2. UC02 - Agendar consulta

O diagrama mostra a consulta aos horários disponíveis e a confirmação do agendamento. Também representa as regras que impedem a reserva simultânea do mesmo horário e o agendamento duplicado para o mesmo médico na mesma data.

```plantuml
@startuml

title UC02 - Agendar consulta - Sequência de projeto
hide footbox
autonumber

actor Paciente
boundary "TelaAgendaMedico" as Tela
control "ConsultaController" as Controller
control "ConsultaService" as Service
database "HorarioAtendimentoRepository" as HorarioRepo
database "ConsultaRepository" as ConsultaRepo
control "LembreteService" as LembreteService
database "LembreteRepository" as LembreteRepo

Paciente -> Tela : selecionarMedico(medicoId, data)
Tela -> Controller : listarHorariosDisponiveis(medicoId, data)
Controller -> Service : consultarHorariosDisponiveis(medicoId, data)
Service -> HorarioRepo : buscarDisponiveisPorMedicoEData(medicoId, data)
HorarioRepo --> Service : horariosDisponiveis
Service --> Controller : horariosDisponiveis
Controller --> Tela : horariosDisponiveis
Tela --> Paciente : exibir agenda e horários livres

alt não existem horários livres
    Tela --> Paciente : informar ausência de horários
else existem horários livres
    Paciente -> Tela : selecionarHorario(horarioId)
    Tela -> Controller : obterResumo(horarioId)
    Controller -> Service : obterResumo(horarioId)
    Service -> HorarioRepo : buscarPorId(horarioId)
    HorarioRepo --> Service : horario
    Service --> Controller : resumoConsulta
    Controller --> Tela : resumoConsulta
    Tela --> Paciente : exibir data, horário, duração e valor

    Paciente -> Tela : confirmarAgendamento()
    Tela -> Controller : agendar(pacienteId, horarioId)
    Controller -> Service : agendar(pacienteId, horarioId)

    Service -> HorarioRepo : bloquearParaReserva(horarioId)
    HorarioRepo --> Service : horarioAtual
    Service -> ConsultaRepo : existeConsultaNaData(pacienteId, medicoId, data)
    ConsultaRepo --> Service : resultado

    alt horário já foi reservado
        Service --> Controller : erroHorarioIndisponivel
        Controller --> Tela : informar indisponibilidade
        Tela --> Paciente : exibir agenda atualizada
    else paciente já possui consulta com o médico na data
        Service -> ConsultaRepo : buscarConsultaExistente(pacienteId, medicoId, data)
        ConsultaRepo --> Service : consultaExistente
        Service --> Controller : erroConsultaDuplicada(consultaExistente)
        Controller --> Tela : apresentar consulta existente
        Tela --> Paciente : informar bloqueio do agendamento
    else horário disponível e sem duplicidade
        Service -> ConsultaRepo : salvar(novaConsulta)
        ConsultaRepo --> Service : consultaAgendada
        Service -> HorarioRepo : marcarComoOcupado(horarioId)
        HorarioRepo --> Service : horário atualizado
        Service -> LembreteService : programarLembrete(consultaAgendada)
        LembreteService -> LembreteRepo : salvar(novoLembrete)
        LembreteRepo --> LembreteService : lembreteProgramado
        LembreteService --> Service : confirmação
        Service --> Controller : consultaAgendada
        Controller --> Tela : confirmação do agendamento
        Tela --> Paciente : exibir confirmação
    end
end

@enduml
```

---

## 3. UC03 - Cancelar consulta pelo paciente

O cancelamento solicitado pelo paciente verifica o prazo mínimo. Quando permitido, a consulta é cancelada, o horário é liberado, o lembrete é cancelado e o médico é notificado.

```plantuml
@startuml

title UC03 - Cancelar consulta pelo paciente - Sequência de projeto
hide footbox
autonumber

actor Paciente
boundary "TelaConsultasPaciente" as Tela
control "ConsultaController" as Controller
control "ConsultaService" as Service
database "ConsultaRepository" as ConsultaRepo
database "HorarioAtendimentoRepository" as HorarioRepo
control "LembreteService" as LembreteService
control "NotificacaoService" as NotificacaoService

Paciente -> Tela : acessarConsultas()
Tela -> Controller : listarConsultasDoPaciente(pacienteId)
Controller -> Service : listarConsultasDoPaciente(pacienteId)
Service -> ConsultaRepo : buscarAgendadasPorPaciente(pacienteId)
ConsultaRepo --> Service : consultas
Service --> Controller : consultas
Controller --> Tela : consultas
Tela --> Paciente : exibir consultas agendadas

Paciente -> Tela : solicitarCancelamento(consultaId)
Tela -> Controller : validarCancelamentoPaciente(consultaId, pacienteId)
Controller -> Service : validarCancelamentoPaciente(consultaId, pacienteId)
Service -> ConsultaRepo : buscarPorIdEParticipante(consultaId, pacienteId)
ConsultaRepo --> Service : consulta

alt consulta dentro do prazo mínimo
    Service --> Controller : cancelamentoNãoPermitido
    Controller --> Tela : restrição de prazo
    Tela --> Paciente : informar que a consulta permanece agendada
else cancelamento permitido
    Service --> Controller : cancelamentoPermitido
    Controller --> Tela : solicitar confirmação
    Tela --> Paciente : exibir confirmação

    alt paciente desiste
        Paciente -> Tela : desistir()
        Tela --> Paciente : manter consulta agendada
    else paciente confirma
        Paciente -> Tela : confirmarCancelamento()
        Tela -> Controller : cancelar(consultaId, pacienteId)
        Controller -> Service : cancelarPorPaciente(consultaId, pacienteId)
        Service -> ConsultaRepo : atualizarStatus(consultaId, CANCELADA)
        ConsultaRepo --> Service : consultaCancelada
        Service -> HorarioRepo : liberar(consulta.horarioId)
        HorarioRepo --> Service : horário liberado
        Service -> LembreteService : cancelarPorConsulta(consultaId)
        LembreteService --> Service : lembrete cancelado
        Service -> NotificacaoService : notificarMedico(consultaCancelada)
        NotificacaoService --> Service : notificação enviada
        Service --> Controller : confirmação
        Controller --> Tela : confirmação
        Tela --> Paciente : informar cancelamento realizado
    end
end

@enduml
```

---

## 4. UC03 - Cancelar consulta pelo médico

O médico não está sujeito ao prazo mínimo definido para o paciente. Após a confirmação, o sistema realiza as mesmas atualizações e notifica o paciente.

```plantuml
@startuml

title UC03 - Cancelar consulta pelo médico - Sequência de projeto
hide footbox
autonumber

actor "Médico" as Medico
boundary "TelaAgendaMedico" as Tela
control "ConsultaController" as Controller
control "ConsultaService" as Service
database "ConsultaRepository" as ConsultaRepo
database "HorarioAtendimentoRepository" as HorarioRepo
control "LembreteService" as LembreteService
control "NotificacaoService" as NotificacaoService

Medico -> Tela : acessarConsultasAgendadas(data)
Tela -> Controller : listarConsultasDoMedico(medicoId, data)
Controller -> Service : listarConsultasDoMedico(medicoId, data)
Service -> ConsultaRepo : buscarAgendadasPorMedicoEData(medicoId, data)
ConsultaRepo --> Service : consultas
Service --> Controller : consultas
Controller --> Tela : consultas
Tela --> Medico : exibir consultas agendadas

Medico -> Tela : solicitarCancelamento(consultaId)
Tela -> Controller : validarCancelamentoMedico(consultaId, medicoId)
Controller -> Service : validarCancelamentoMedico(consultaId, medicoId)
Service -> ConsultaRepo : buscarPorIdEParticipante(consultaId, medicoId)
ConsultaRepo --> Service : consulta
Service --> Controller : cancelamentoPermitido
Controller --> Tela : solicitar confirmação
Tela --> Medico : exibir confirmação

alt médico desiste
    Medico -> Tela : desistir()
    Tela --> Medico : manter consulta agendada
else médico confirma
    Medico -> Tela : confirmarCancelamento()
    Tela -> Controller : cancelar(consultaId, medicoId)
    Controller -> Service : cancelarPorMedico(consultaId, medicoId)
    Service -> ConsultaRepo : atualizarStatus(consultaId, CANCELADA)
    ConsultaRepo --> Service : consultaCancelada
    Service -> HorarioRepo : liberar(consulta.horarioId)
    HorarioRepo --> Service : horário liberado
    Service -> LembreteService : cancelarPorConsulta(consultaId)
    LembreteService --> Service : lembrete cancelado
    Service -> NotificacaoService : notificarPaciente(consultaCancelada)
    NotificacaoService --> Service : notificação enviada
    Service --> Controller : confirmação
    Controller --> Tela : confirmação
    Tela --> Medico : informar cancelamento realizado
end

@enduml
```

---

## 5. UC06 - Manter agenda de atendimento

O diagrama representa a inclusão, a alteração e a remoção de períodos da agenda do médico. Antes de persistir uma mudança, o serviço verifica sobreposição e consultas já agendadas.

```plantuml
@startuml

title UC06 - Manter agenda de atendimento - Sequência de projeto
hide footbox
autonumber

actor "Médico" as Medico
boundary "TelaConfiguracaoAgenda" as Tela
control "AgendaController" as Controller
control "AgendaService" as Service
database "AgendaRepository" as AgendaRepo
database "HorarioAtendimentoRepository" as HorarioRepo
database "ConsultaRepository" as ConsultaRepo

Medico -> Tela : acessarAgenda()
Tela -> Controller : obterAgenda(medicoId)
Controller -> Service : obterAgenda(medicoId)
Service -> AgendaRepo : buscarPorMedico(medicoId)
AgendaRepo --> Service : agenda
Service -> HorarioRepo : buscarPeriodos(agendaId)
HorarioRepo --> Service : períodos
Service --> Controller : agendaComPeriodos
Controller --> Tela : agendaComPeriodos
Tela --> Medico : exibir períodos cadastrados

Medico -> Tela : salvarPeriodo(dadosPeriodo, operação)
Tela -> Controller : salvarPeriodo(medicoId, dadosPeriodo, operação)
Controller -> Service : salvarPeriodo(medicoId, dadosPeriodo, operação)
Service -> HorarioRepo : existeSobreposição(agendaId, dadosPeriodo)
HorarioRepo --> Service : resultadoSobreposição

alt período se sobrepõe a outro
    Service --> Controller : erroSobreposição
    Controller --> Tela : conflito de horários
    Tela --> Medico : informar que a alteração não foi realizada
else sem sobreposição
    Service -> ConsultaRepo : buscarConsultasAfetadas(dadosPeriodo)
    ConsultaRepo --> Service : consultasAfetadas

    alt alteração ou remoção afeta consultas agendadas
        Service --> Controller : erroConsultasAfetadas(consultasAfetadas)
        Controller --> Tela : consultasAfetadas
        Tela --> Medico : exigir tratamento das consultas
    else dados válidos e sem consultas afetadas
        alt operação = incluir
            Service -> HorarioRepo : salvar(novoPeriodo)
        else operação = alterar
            Service -> HorarioRepo : atualizar(periodo)
        else operação = remover
            Service -> HorarioRepo : remover(periodoId)
        end
        HorarioRepo --> Service : período atualizado
        Service --> Controller : agendaAtualizada
        Controller --> Tela : agendaAtualizada
        Tela --> Medico : exibir confirmação e nova agenda
    end
end

@enduml
```

---

## 6. UC07 - Consultar pacientes do dia

O diagrama mostra a montagem do painel diário do médico, incluindo pacientes agendados, horários livres, previsão de horas de atendimento e valor previsto a receber.

```plantuml
@startuml

title UC07 - Consultar pacientes do dia - Sequência de projeto
hide footbox
autonumber

actor "Médico" as Medico
boundary "TelaPainelDiario" as Tela
control "PainelMedicoController" as Controller
control "PainelMedicoService" as Service
database "ConsultaRepository" as ConsultaRepo
database "HorarioAtendimentoRepository" as HorarioRepo

Medico -> Tela : acessarPainel(data)
Tela -> Controller : consultarPainel(medicoId, data)
Controller -> Service : montarPainel(medicoId, data)
Service -> ConsultaRepo : buscarAgendadasPorMedicoEData(medicoId, data)
ConsultaRepo --> Service : consultas
Service -> HorarioRepo : buscarPorMedicoEData(medicoId, data)
HorarioRepo --> Service : horários
Service -> Service : identificarHorariosLivres(horários, consultas)

alt não existem consultas agendadas
    Service --> Controller : painelSemConsultas(horáriosLivres)
    Controller --> Tela : painelSemConsultas
    Tela --> Medico : informar ausência de consultas e agenda livre
else existem consultas agendadas
    Service -> Service : calcularHorasPrevistas(consultas)

    alt médico informou valor dos atendimentos
        Service -> Service : calcularValorPrevisto(consultas)
        Service --> Controller : painel(consultas, horáriosLivres, horas, valor)
    else médico não informou valor
        Service --> Controller : painel(consultas, horáriosLivres, horas)
    end

    Controller --> Tela : dadosDoPainel
    Tela --> Medico : exibir pacientes, horários e previsões
end

Medico -> Tela : selecionarOutraData(novaData)
Tela -> Controller : consultarPainel(medicoId, novaData)
Controller -> Service : montarPainel(medicoId, novaData)
Service --> Controller : dadosDaNovaData
Controller --> Tela : dadosDaNovaData
Tela --> Medico : atualizar painel

@enduml
```

---

## 7. Pendências para a versão final

Antes de inserir os diagramas no documento consolidado, será necessário:

1. conferir os nomes das classes e dos métodos no diagrama de classes de projeto;
2. substituir os nomes genéricos caso o grupo adote convenções específicas de algum framework;
3. confirmar se lembretes e notificações serão serviços internos ou integrações externas;
4. renderizar os diagramas em alta resolução;
5. numerar as figuras e incluir suas referências no texto e na lista de figuras.

