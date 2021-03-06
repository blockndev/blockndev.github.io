---
title: "Хэшировать ли публичные ключи в Биткоин-адресах?"
date: 2020-04-11T00:00:00+00:00
categories: [Биткоин, Криптография]
---

_Как вы знаете, Биткоин-адрес это хэш публичного ключа, закодированный в Base58 (за исключением bech32-адресов) для более удобного чтения. Когда вы переводите биткоины на адрес, вы переводите их именно на хэш публичного ключа, а когда вы выводите их с адреса, вы должны раскрыть сам публичный ключ. Но так было не всегда. В самой первой версии Биткоина, выпущенной Сатоши, перевод происходил на публичный ключ. Это считалось небезопасным, поэтому было сделано хэширование ключей. Но действительно ли открытые публичные ключи это такая проблема? Предлагаю вам прочитать мнение Питера Вулле (Pieter Wuille) на этот счёт._


## Теория

Считается, что, чтобы подделать ECDSA-подпись, имея публичный ключ, нужно вычислить приватный ключ для него (эта операция называется дискретным логарифмом — сложность её вычисления и лежит в основе надежности ECDSA). И для этого нужно сначала получить публичный ключ.

Имея публичный ключ, вам нужно совершить 2128 операций, чтобы найти соответствующий ему приватный. Это просто чудовищное количество работы (даже если каждый компьютер в мире сможет делать одну операцию за один такт процессора, то потребуется более 100 миллионов лет; на практике потребуется на несколько порядков больше времени). Однако эти вычисления не учитывают развития алгоритмов решения задачи дискретного логарифма и развития квантовых компьютеров. Достаточно мощный квантовый компьютер (ничего близкого к такому пока не существует) сможет решить эту задачу _намного_ быстрее.

Используя в адресах хэш публичного ключа, а не сам публичны ключ, публичный ключ становится известным всем только в момент расходования выхода транзакции. Если не учитывать (невероятные) уязвимости, найденные в используемых хэш-функциях (SHA256 и RIPEMD160), то восстановление ключа из хэша это непростая задача даже для квантовых компьютеров. Но все же 160-битные хэши считаются относительно слабыми (для их взлома требуется 280 операций на квантовом компьютере).

Проще говоря, хэширование публичных ключей в адресах сильно затрудняет кражу монет на квантовом компьютере или через взлом дискретных логарифмов.

## Практика

В этом разделе я выскажу своё мнение, с которым некоторые могут не согласиться.

Я считаю, что польза от хэширования публичных ключей в лучшем случае незначительна. В худшем случае хэширование создаёт ложное ощущение безопасности. На это есть несколько причин:
1. Польза от хэширования есть только до момента расходования выхода траназакции. Как только кто-то попытается вывести биткоины с P2PKH-выхода (Pay-to-public-key-hash, оплата на хэш публичного ключа), публичный ключ будет раскрыт. Сговорившись с майнерами, злоумышленник может затормозить добычу транзакции, попытаться восстановить приватный ключ из раскрытого публичного с помощью квантового компьютера и украсть монеты.
2. Люди продолжают использовать одни и те же адреса, и этого трудно избежать. Если адрес переиспользуется, то в момент первого расходования публичный ключ становится известен всем, его хэширование больше не помогает.
3. Почти все самые интересные варианты использования Биткоина (мультиподписи, 2FA, эскроу, платёжные каналы, BIP32-аккаунты) требуют обмена публичными ключами друг с другом. Считать, что в таких сценариях хэширование публичных ключей даёт какую-то защиту, было бы самообманом: публичные ключи и так постоянно раскрываются, даже если пользователи об этом не подозревают.
4. Даже если вы будете строго следить за тем, чтобы не раскрывать свои публичные ключи и не пользоваться вышеприведёнными механиками, в блокчейне Биткоина хранится более 5 миллионов биткоинов (по моим расчетам) с открытыми публичными ключами. Не могу себе представить, что биткоин будет иметь хоть какую-то стоимость, если кому-то удасться украсть эти 5 миллионов.

Но это не значит, что у нас есть проблемы. До создания достаточно мощных квантовых компьютеров ещё очень далеко (если вообще возможно создать квантовые компьютеры с таким гигантским количеством q-битов, которые необходимы для решения дискретных логарифмов). И это даёт нам время перейти на алгоритмы, стойкие к квантовым вычислениям (перестать использовать ECDSA и похожие алгоритмы). Этот переход ещё пока не сделан, так как у существующих алгоритмов, устойчивых к квантовым вычислениям, есть несколько недостатков, таких как очень длинные ключи и подписи. Из-за этого они пока очень непопулярны, но исследования продолжаются, и если они понадобятся, то они есть.

## Должны ли мы использовать публичные ключи в качестве адресов?

_Вопрос: если создавать новую криптовалюту, то стоит ли сразу использовать закодированные в Base58 публичные адреса с чексуммой?_

Тапрут именно это и делает. Тапрут-выходы (а значит и адреса) содержат полные публичные ключи — у этого есть несколько преимуществ, например: они короче, дешевле и позволяют создавать поверх них более сложные протоколы. Для таких выходов используется формат адресов Bech32, у которого тоже есть несколько преимуществ над Base58: проще конвертировать в удобную для чтения форму и сравнивать друг с другом, более надежная проверка на ошибки, формат более расширяемый, меньший размер QR-кодов и т. д.

Дисклаймер: я соавтор протокола Тапрут и стандарта Bech32.

**TL;DR публичные ключи должны быть публичными.**

Оригинал: https://bitcoin.stackexchange.com/a/95133