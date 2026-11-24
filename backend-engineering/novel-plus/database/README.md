# Database

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [novel-plus overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. Database Topology](#1-database-topology)
> - [2. Tables](#2-tables)
> - [3. book_content Sharding](#3-book_content-sharding)

## 0. Overview

> The MySQL database structure used by Novel Plus.

1. [Database Topology](#1-database-topology): All application reads and writes use one <code>novel_plus</code> MySQL database through ShardingSphere-JDBC.
2. [Tables](#2-tables): Novel Plus uses 40 logical tables; only <code>book_content</code> is sharded.
3. [book_content Sharding](#3-book_content-sharding): <code>book_content</code> is routed across ten physical tables by ShardingSphere-JDBC.
4. [MyBatis Data Access](01-mybatis-data-access.md): MyBatis converts Mapper or DAO method calls into SQL before execution through ShardingSphere-JDBC.

## 1. Database Topology

> All application reads and writes use one <code>novel_plus</code> MySQL database through ShardingSphere-JDBC.

```mermaid
flowchart LR
    A[Applications] --> S[ShardingSphere-JDBC]
    S --> D[(novel_plus MySQL)]
```

| Setting | Current configuration |
| --- | --- |
| Application database | <code>novel_plus</code> |
| Application data source | <code>ds_1</code> |
| Writer nodes | 1 |
| Read replicas | 0 |
| Read/write splitting | Not configured |
| Connection layer | ShardingSphere-JDBC |
| Sharding rule | <code>config/shardingsphere-jdbc.yml</code> |
| Application JDBC URL | <code>jdbc:shardingsphere:absolutepath:${user.dir}/config/shardingsphere-jdbc.yml</code> |
| <code>information_schema</code> | MySQL metadata; not a replica |

## 2. Tables

> Novel Plus uses 40 logical tables; only <code>book_content</code> is sharded.

<table>
  <thead>
    <tr>
      <th>Group</th>
      <th>Logical table</th>
      <th>Sharding</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="9">Novel content</td>
      <td><code>book</code></td>
      <td>—</td>
    </tr>
    <tr><td><code>book_author</code></td><td>—</td></tr>
    <tr><td><code>book_category</code></td><td>—</td></tr>
    <tr><td><code>book_comment</code></td><td>—</td></tr>
    <tr><td><code>book_comment_reply</code></td><td>—</td></tr>
    <tr>
      <td><code>book_content</code></td>
      <td><a href="#3-book_content-sharding"><code>book_content0</code>–<code>book_content9</code></a><br><code>index_id % 10</code></td>
    </tr>
    <tr><td><code>book_index</code></td><td>—</td></tr>
    <tr><td><code>book_screen_bullet</code></td><td>—</td></tr>
    <tr><td><code>book_setting</code></td><td>—</td></tr>
    <tr>
      <td rowspan="4">Authors</td>
      <td><code>author</code></td>
      <td>—</td>
    </tr>
    <tr><td><code>author_code</code></td><td>—</td></tr>
    <tr><td><code>author_income</code></td><td>—</td></tr>
    <tr><td><code>author_income_detail</code></td><td>—</td></tr>
    <tr>
      <td rowspan="6">Users and payments</td>
      <td><code>user</code></td>
      <td>—</td>
    </tr>
    <tr><td><code>user_bookshelf</code></td><td>—</td></tr>
    <tr><td><code>user_buy_record</code></td><td>—</td></tr>
    <tr><td><code>user_feedback</code></td><td>—</td></tr>
    <tr><td><code>user_read_history</code></td><td>—</td></tr>
    <tr><td><code>order_pay</code></td><td>—</td></tr>
    <tr>
      <td rowspan="3">Crawler</td>
      <td><code>crawl_batch_task</code></td>
      <td>—</td>
    </tr>
    <tr><td><code>crawl_single_task</code></td><td>—</td></tr>
    <tr><td><code>crawl_source</code></td><td>—</td></tr>
    <tr>
      <td rowspan="4">Website content</td>
      <td><code>friend_link</code></td>
      <td>—</td>
    </tr>
    <tr><td><code>news</code></td><td>—</td></tr>
    <tr><td><code>news_category</code></td><td>—</td></tr>
    <tr><td><code>website_info</code></td><td>—</td></tr>
    <tr>
      <td>Administration</td>
      <td><code>sys_*</code> (14 admin-system tables)</td>
      <td>—</td>
    </tr>
  </tbody>
</table>

## 3. book_content Sharding

> <code>book_content</code> is routed across ten physical tables by ShardingSphere-JDBC.

1. Storage Mode

   | Mode | Content location | Sharding |
   | --- | --- | --- |
   | <code>db</code> | <code>book_content0</code>–<code>book_content9</code> | Yes |
   | <code>txt</code> | Filesystem at <code>content.save.path</code> | No |

   - Crawler writes follow <code>content.save.storage</code>.
   - Frontend reads follow each chapter's <code>book_index.storage_type</code>.

2. Sharding Rule
   - Logical table: <code>book_content</code>
   - Shard key: <code>index_id</code>
   - Physical tables: <code>book_content0</code>–<code>book_content9</code>
   - Rule: <code>book_content${index_id % 10}</code>

3. <a name="direct-sql"></a>Direct SQL

   <table>
     <thead>
       <tr>
         <th>Module</th>
         <th>Direct SQL method</th>
         <th>SQL pattern</th>
         <th>Shard routing</th>
       </tr>
     </thead>
     <tbody>
       <tr>
         <td rowspan="4"><code>novel-front</code></td>
         <td><code>BookContentMapper.selectMany</code></td>
         <td><code>SELECT ... WHERE index_id = ?</code></td>
         <td>One shard</td>
       </tr>
       <tr>
         <td><code>BookContentMapper.insertSelective</code></td>
         <td><code>INSERT ... index_id</code></td>
         <td>One shard</td>
       </tr>
       <tr>
         <td><code>BookContentMapper.update</code></td>
         <td><code>UPDATE ... WHERE index_id = ?</code></td>
         <td>One shard</td>
       </tr>
       <tr>
         <td><code>BookContentMapper.delete</code></td>
         <td><code>DELETE ... WHERE index_id = ?</code></td>
         <td>One shard</td>
       </tr>
       <tr>
         <td rowspan="5"><code>novel-admin</code></td>
         <td><code>BookContentDao.list</code><br><code>BookContentDao.count</code></td>
         <td>Optional <code>WHERE index_id = ?</code></td>
         <td>One shard when present;<br>all shards when absent</td>
       </tr>
       <tr>
         <td><code>BookContentDao.get</code><br><code>BookContentDao.update</code><br><code>BookContentDao.remove</code></td>
         <td><code>WHERE id = ?</code></td>
         <td>All shards</td>
       </tr>
       <tr>
         <td><code>BookContentDao.batchRemove</code></td>
         <td><code>WHERE id IN (...)</code></td>
         <td>All shards</td>
       </tr>
       <tr>
         <td><code>BookContentDao.save</code></td>
         <td><code>INSERT ... index_id</code></td>
         <td>One shard</td>
       </tr>
       <tr>
         <td><code>BookContentDao.listByIndexIds</code><br><code>BookContentDao.removeByIndexIds</code></td>
         <td><code>WHERE index_id IN (...)</code></td>
         <td>Matching shards</td>
       </tr>
       <tr>
         <td rowspan="2"><code>novel-crawl</code></td>
         <td><code>BookContentMapper.insertMultiple</code><br><code>BookContentMapper.insertSelective</code></td>
         <td><code>INSERT ... index_id</code></td>
         <td>One shard per row</td>
       </tr>
       <tr>
         <td><code>BookContentMapper.update</code></td>
         <td><code>UPDATE ... WHERE index_id = ?</code></td>
         <td>One shard</td>
       </tr>
     </tbody>
   </table>
