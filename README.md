 Data Fundamentals Project: Admin Roles & Security in Supabase

 Overview

This project was developed as part of the **Data fundamentals unit**.
The goal was to implement **Admin and User roles**, **Row Level Security (RLS)**, and **data access policies** using **Supabase (PostgreSQL)**.

The database models a simple **real estate system** with clients, properties, sales, and users.
Each user has restricted access to their own data, while admins can manage all records.

---

 Database Schema

The database includes four main tables:

| Table      | Description                                        |
| -----------| -------------------------------------------------- |
| users      | Stores system users with roles (`admin` or `user`) |
| clients    | Holds client information and preferences           |
| properties | Contains property listings                         |
| sales      | Records sales transactions                         |

Each table has an `owner` column (UUID) linked to the `users` table to enforce ownership rules.

---

Setup Steps

1.Enable Row Level Security (RLS)

RLS was enabled for all tables to restrict access based on user identity.

```sql
alter table public.clients enable row level security;
alter table public.properties enable row level security;
alter table public.sales enable row level security;
alter table public.users enable row level security;
```

---

 2. Define Roles

Two roles were used in this project:

| Role  | Description                                |
| ----- | ------------------------------------------ |
| Admin | Full privileges on all tables              |
| User  | Can only view and insert their own records |

---

 3.Access Control Policies

 Clients

```sql
create policy "Users can view their own clients"
on public.clients
for select
using (auth.uid() = owner);

create policy "Users can insert their own clients"
on public.clients
for insert
with check (auth.uid() = owner);

create policy "Admins have full access to clients"
on public.clients
for all
using (exists (select 1 from public.users where id = auth.uid() and role = 'admin'));
```

 Properties

```sql
create policy "Users can view their own properties"
on public.properties
for select
using (auth.uid() = owner);

create policy "Users can insert their own properties"
on public.properties
for insert
with check (auth.uid() = owner);

create policy "Admins have full access to properties"
on public.properties
for all
using (exists (select 1 from public.users where id = auth.uid() and role = 'admin'));
```

Sales

```sql
create policy "Users can view their own sales"
on public.sales
for select
using (auth.uid() = owner);

create policy "Users can insert their own sales"
on public.sales
for insert
with check (auth.uid() = owner);

create policy "Admins have full access to sales"
on public.sales
for all
using (exists (select 1 from public.users where id = auth.uid() and role = 'admin'));
```

 Users

```sql
create policy "Users can view their own profile"
on public.users
for select
using (auth.uid() = id);

create policy "Admins can manage all users"
on public.users
for all
using (exists (select 1 from public.users where id = auth.uid() and role = 'admin'));
```



 4.Admin-Only Function

A secure function allows only admins to delete property records.

````sql
create or replace function delete_property(property_id integer)
returns void
language sql
security definer
as $$
  delete from public.properties where property_id = delete_property.property_id;
$$;


Security Model Summary

| Component       | Description                                    |
| ----------------| ---------------------------------------------- |
| Authentication | Managed by Supabase Auth (email/password)      |
| Authorization | Enforced by RLS and SQL policies               |
| Access Control | Role-based: Admins vs Users                    |
| Least Privilege | Users can only interact with their own records |

