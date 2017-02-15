---
title: Руководство - Типы данных
---

# Типы данных

Типы данных позволяют определить [структуру сущностей](/guides/models/entities#custom-schema), которую не всегда необходимо задавать вручную.

Для сущностей есть 5 типов данных:

  * **Definition** — базовые типы данных Ruby;
  * **Strict** — базовые типы с простой проверкой входных данных;
  * **Coercible** — типы, конструктор которых способен преобразовать входные данные к заданному типу;
  * **Form** — типы, конструктор которых способен преобразовать данные формата характерного передаваемым через HTTP формам;
  * **JSON** — типы, конструктор которых способен преобразовать данные формата JSON.

## Definition

  * `Types::Nil`
  * `Types::String`
  * `Types::Symbol`
  * `Types::Int`
  * `Types::Float`
  * `Types::Decimal`
  * `Types::Class`
  * `Types::Bool`
  * `Types::True`
  * `Types::False`
  * `Types::Date`
  * `Types::DateTime`
  * `Types::Time`
  * `Types::Array`
  * `Types::Hash`

## Strict

  * `Types::Strict::Nil`
  * `Types::Strict::String`
  * `Types::Strict::Symbol`
  * `Types::Strict::Int`
  * `Types::Strict::Float`
  * `Types::Strict::Decimal`
  * `Types::Strict::Class`
  * `Types::Strict::Bool`
  * `Types::Strict::True`
  * `Types::Strict::False`
  * `Types::Strict::Date`
  * `Types::Strict::DateTime`
  * `Types::Strict::Time`
  * `Types::Strict::Array`
  * `Types::Strict::Hash`

## Coercible

  * `Types::Coercible::String`
  * `Types::Coercible::Int`
  * `Types::Coercible::Float`
  * `Types::Coercible::Decimal`
  * `Types::Coercible::Array`
  * `Types::Coercible::Hash`

## Form

  * `Types::Form::Nil`
  * `Types::Form::Int`
  * `Types::Form::Float`
  * `Types::Form::Decimal`
  * `Types::Form::Bool`
  * `Types::Form::True`
  * `Types::Form::False`
  * `Types::Form::Date`
  * `Types::Form::DateTime`
  * `Types::Form::Time`
  * `Types::Form::Array`
  * `Types::Form::Hash`

## JSON

  * `Types::Json::Nil`
  * `Types::Json::Decimal`
  * `Types::Json::Date`
  * `Types::Json::DateTime`
  * `Types::Json::Time`
  * `Types::Json::Array`
  * `Types::Json::Hash`

---

Типы данных в моделях Hanami основаны на геме`dry-types`. Подробнее о них: [http://dry-rb.org/gems/dry-types](http://dry-rb.org/gems/dry-types)
