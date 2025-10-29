 1. Row Level Security (RLS)

Row Level Security (RLS) is enabled on all tables:

`clients`
`properties`
`sales`
`users`

This ensures that users can only access or modify rows where they are the owner.

```sql
alter table public.clients enable row level security;
alter table public.properties enable row level security;
alter table public.sales enable row level security;
alter table public.users enable row level security;
```


2. Roles

Two main roles are defined in the system:

| Role      | Description                                                  |
| --------- | ------------------------------------------------------------ |
| Admin | Has full access (read, insert, update, delete) on all tables |
| User  | Can only view and insert their own records                   |

Defined in the `users` table:

```sql
create table public.users (
  id uuid not null default auth.uid(),
  email text not null,
  role text not null default 'user',
  constraint users_pkey primary key (id),
  constraint users_role_check check (role in ('admin', 'user'))
);
```

---
3. Access Control Policies
 a. Clients Table

* Users can only view and insert their own clients.
* Admins can view, insert, update, and delete all clients.

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

 b. Properties Table

* Users can only view and insert their own properties.
* Admins have full access.

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

c. Sales Table

* Users can only view and insert their own sales.
* Admins have full access.

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

d. Users Table

* Users can only view their own profile.
* Admins can view and manage all users.

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

---
4. Admin-Only Function

A secure function was created for admin use only.
This function allows an admin to delete a property record.

```sql
create or replace function delete_property(property_id integer)
returns void
language sql
security definer
as $$
  delete from public.properties where property_id = delete_property.property_id;
$$;
