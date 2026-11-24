# MongoDB

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [mall overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. Collections](#1-collections)

## 0. Overview

> MongoDB stores three denormalized member-activity collections used only by `mall-portal`.

1. [Collections](#1-collections): The collections store browsing history, product collections, and brand attention in the `mall-port` database.
2. Architecture
   - One standalone MongoDB 4 server with no replica set or sharding.
   - Spring Data derives the collection names because the `@Document` annotations do not specify them explicitly.
   - Each document copies member and product or brand fields from MySQL when it is created; later MySQL changes are not synchronized automatically.

## 1. Collections

> The collections store browsing history, product collections, and brand attention in the `mall-port` database.

| Collection | Java document | Purpose | Indexed fields |
| --- | --- | --- | --- |
| `memberReadHistory` | `MemberReadHistory` | Product browsing history | `memberId`, `productId` |
| `memberProductCollection` | `MemberProductCollection` | Favorited products | `memberId`, `productId` |
| `memberBrandAttention` | `MemberBrandAttention` | Followed brands | `memberId`, `brandId` |
