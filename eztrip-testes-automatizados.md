# EzTrip: Testes Automatizados Existentes

## Objetivo

Este documento resume os testes automatizados encontrados no repositório e explica como eles ajudaram a consolidar os cenários manuais do TCC.

## Backend

### Testes de integração

Foram encontrados testes cobrindo autenticação:

- cadastro
- bloqueio de cadastro duplicado
- login com e-mail não verificado
- confirmação de e-mail
- reenvio de verificação
- login com credenciais incorretas
- refresh token
- logout
- recuperação e redefinição de senha
- bloqueio de reuso de token de redefinição

### Testes unitários

Foram encontrados testes para:

- hash e verificação de senha com Argon2
- regras arquiteturais entre camadas

## Frontend

Foram encontrados testes de interface e fluxo para páginas como:

- login
- cadastro
- verificação de e-mail
- esqueci minha senha
- redefinição de senha
- lista de viagens
- criação de viagem
- lista de bagagem

## Como os testes automatizados ajudaram esta documentação

Os testes automatizados foram usados como apoio para confirmar:

- regras de autenticação e mensagens de erro
- fluxo de confirmação de e-mail
- fluxo de recuperação de senha
- comportamento esperado da criação de viagem
- operações permitidas na lista de bagagem

## Limite desta análise

Esta documentação foi derivada da leitura do código, dos testes existentes e das configurações do projeto. Não houve execução local dos testes automatizados nesta etapa de documentação.
