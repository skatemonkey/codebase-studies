# MySQL

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [mall overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. Tables](#1-tables)

## 0. Overview

> MySQL stores Mall's core commerce and administration data in one `mall` schema containing 76 tables.

1. [Tables](#1-tables): The schema groups its 76 tables under five domain prefixes.
2. Architecture
   - One standalone MySQL 5.7 server.
   - No replicas, sharding, or read/write splitting.
   - The SQL schema declares no foreign-key constraints; relationships are maintained by IDs and application logic.

## 1. Tables

> The schema groups its 76 tables under five domain prefixes.

<table>
  <thead>
    <tr><th>Group</th><th>Table</th><th>Purpose</th></tr>
  </thead>
  <tbody>
    <tr><td rowspan="12">Content · <code>cms</code></td><td><code>cms_help</code></td><td>Help articles</td></tr>
    <tr><td><code>cms_help_category</code></td><td>Help categories</td></tr>
    <tr><td><code>cms_member_report</code></td><td>Member reports</td></tr>
    <tr><td><code>cms_prefrence_area</code></td><td>Curated product areas</td></tr>
    <tr><td><code>cms_prefrence_area_product_relation</code></td><td>Curated-area product relations</td></tr>
    <tr><td><code>cms_subject</code></td><td>Featured subjects</td></tr>
    <tr><td><code>cms_subject_category</code></td><td>Subject categories</td></tr>
    <tr><td><code>cms_subject_comment</code></td><td>Subject comments</td></tr>
    <tr><td><code>cms_subject_product_relation</code></td><td>Subject-product relations</td></tr>
    <tr><td><code>cms_topic</code></td><td>Community topics</td></tr>
    <tr><td><code>cms_topic_category</code></td><td>Topic categories</td></tr>
    <tr><td><code>cms_topic_comment</code></td><td>Topic comments</td></tr>
    <tr><td rowspan="8">Orders · <code>oms</code></td><td><code>oms_cart_item</code></td><td>Shopping-cart items</td></tr>
    <tr><td><code>oms_company_address</code></td><td>Company shipping and receiving addresses</td></tr>
    <tr><td><code>oms_order</code></td><td>Orders</td></tr>
    <tr><td><code>oms_order_item</code></td><td>Order items</td></tr>
    <tr><td><code>oms_order_operate_history</code></td><td>Order-operation history</td></tr>
    <tr><td><code>oms_order_return_apply</code></td><td>Return applications</td></tr>
    <tr><td><code>oms_order_return_reason</code></td><td>Return reasons</td></tr>
    <tr><td><code>oms_order_setting</code></td><td>Order settings</td></tr>
    <tr><td rowspan="18">Products · <code>pms</code></td><td><code>pms_album</code></td><td>Product albums</td></tr>
    <tr><td><code>pms_album_pic</code></td><td>Album images</td></tr>
    <tr><td><code>pms_brand</code></td><td>Brands</td></tr>
    <tr><td><code>pms_comment</code></td><td>Product reviews</td></tr>
    <tr><td><code>pms_comment_replay</code></td><td>Product-review replies</td></tr>
    <tr><td><code>pms_feight_template</code></td><td>Freight templates</td></tr>
    <tr><td><code>pms_member_price</code></td><td>Member-specific product prices</td></tr>
    <tr><td><code>pms_product</code></td><td>Products</td></tr>
    <tr><td><code>pms_product_attribute</code></td><td>Product attributes</td></tr>
    <tr><td><code>pms_product_attribute_category</code></td><td>Product-attribute categories</td></tr>
    <tr><td><code>pms_product_attribute_value</code></td><td>Product-attribute values</td></tr>
    <tr><td><code>pms_product_category</code></td><td>Product categories</td></tr>
    <tr><td><code>pms_product_category_attribute_relation</code></td><td>Category-attribute relations</td></tr>
    <tr><td><code>pms_product_full_reduction</code></td><td>Full-reduction product discounts</td></tr>
    <tr><td><code>pms_product_ladder</code></td><td>Quantity-based product pricing</td></tr>
    <tr><td><code>pms_product_operate_log</code></td><td>Product-change history</td></tr>
    <tr><td><code>pms_product_vertify_record</code></td><td>Product-approval records</td></tr>
    <tr><td><code>pms_sku_stock</code></td><td>SKU inventory</td></tr>
    <tr><td rowspan="13">Promotions · <code>sms</code></td><td><code>sms_coupon</code></td><td>Coupons</td></tr>
    <tr><td><code>sms_coupon_history</code></td><td>Coupon receipt and usage history</td></tr>
    <tr><td><code>sms_coupon_product_category_relation</code></td><td>Coupon-category relations</td></tr>
    <tr><td><code>sms_coupon_product_relation</code></td><td>Coupon-product relations</td></tr>
    <tr><td><code>sms_flash_promotion</code></td><td>Flash promotions</td></tr>
    <tr><td><code>sms_flash_promotion_log</code></td><td>Flash-promotion notifications</td></tr>
    <tr><td><code>sms_flash_promotion_product_relation</code></td><td>Flash-promotion product relations</td></tr>
    <tr><td><code>sms_flash_promotion_session</code></td><td>Flash-promotion sessions</td></tr>
    <tr><td><code>sms_home_advertise</code></td><td>Homepage advertisements</td></tr>
    <tr><td><code>sms_home_brand</code></td><td>Homepage brand recommendations</td></tr>
    <tr><td><code>sms_home_new_product</code></td><td>Homepage new products</td></tr>
    <tr><td><code>sms_home_recommend_product</code></td><td>Homepage product recommendations</td></tr>
    <tr><td><code>sms_home_recommend_subject</code></td><td>Homepage subject recommendations</td></tr>
    <tr><td rowspan="25">Users and access · <code>ums</code></td><td><code>ums_admin</code></td><td>Administration users</td></tr>
    <tr><td><code>ums_admin_login_log</code></td><td>Administration login history</td></tr>
    <tr><td><code>ums_admin_permission_relation</code></td><td>Administration user-permission relations</td></tr>
    <tr><td><code>ums_admin_role_relation</code></td><td>Administration user-role relations</td></tr>
    <tr><td><code>ums_growth_change_history</code></td><td>Member growth-value history</td></tr>
    <tr><td><code>ums_integration_change_history</code></td><td>Member points history</td></tr>
    <tr><td><code>ums_integration_consume_setting</code></td><td>Points-consumption settings</td></tr>
    <tr><td><code>ums_member</code></td><td>Members</td></tr>
    <tr><td><code>ums_member_level</code></td><td>Member levels</td></tr>
    <tr><td><code>ums_member_login_log</code></td><td>Member login history</td></tr>
    <tr><td><code>ums_member_member_tag_relation</code></td><td>Member-tag relations</td></tr>
    <tr><td><code>ums_member_product_category_relation</code></td><td>Member product-category preferences</td></tr>
    <tr><td><code>ums_member_receive_address</code></td><td>Member delivery addresses</td></tr>
    <tr><td><code>ums_member_rule_setting</code></td><td>Member points and growth rules</td></tr>
    <tr><td><code>ums_member_statistics_info</code></td><td>Member statistics</td></tr>
    <tr><td><code>ums_member_tag</code></td><td>Member tags</td></tr>
    <tr><td><code>ums_member_task</code></td><td>Member tasks</td></tr>
    <tr><td><code>ums_menu</code></td><td>Administration menus</td></tr>
    <tr><td><code>ums_permission</code></td><td>Administration permissions</td></tr>
    <tr><td><code>ums_resource</code></td><td>Protected administration resources</td></tr>
    <tr><td><code>ums_resource_category</code></td><td>Protected-resource categories</td></tr>
    <tr><td><code>ums_role</code></td><td>Administration roles</td></tr>
    <tr><td><code>ums_role_menu_relation</code></td><td>Role-menu relations</td></tr>
    <tr><td><code>ums_role_permission_relation</code></td><td>Role-permission relations</td></tr>
    <tr><td><code>ums_role_resource_relation</code></td><td>Role-resource relations</td></tr>
  </tbody>
</table>
