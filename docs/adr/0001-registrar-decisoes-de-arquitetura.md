# 1. Registrar decisões de arquitetura

Data: 26/11/2026

## Status

Aceita

## Contexto

O Ninho exercita de propósito padrões que o volume dele não exige, e o PRD pede que cada um tenha uma versão simplificada comparada em ADR. As decisões precisam ficar versionadas junto da documentação, com o motivo de cada escolha.

## Decisão

Registrar cada decisão de arquitetura significativa como um ADR no formato de Michael Nygard, em `docs/adr/`, numerado em sequência (`NNNN-titulo-curto.md`). Um ADR aceito não é editado; uma mudança de decisão vira um ADR novo que substitui o anterior.

## Consequências

O histórico das decisões fica no repositório e passa por pull request. Os ADRs previstos no PRD (hexagonal, fila, conflito offline, extração de Notificações, versão completa × simplificada, detecção de falhas, front) entram a partir do F1 M2.
