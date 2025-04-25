---
title: "SQLcl Connection Manager - Folders"
date: 2025-04-23
layout: post
tags: sqlcl oracle
---

In SQLcl version 25 the connection manager now includes support for folders.
Let's see how this works, we'll be using the six connections configured below:

```sql
SQL> cm list
.
├── dick
├── harriet
├── harry
├── claire
├── marie
└── sophie
SQL>
```

Let's separate the boys from the girls.  To do this, we need to add the folders first, then move each connection.

```sql
SQL> cm add -folder boys
Folder boys has been added
SQL> cm move -conn tom boys
Connection tom has been moved to boys
SQL> cm move -conn dick  boys
Connection dick has been moved to boys
SQL> cm move -conn harry boys
Connection harry has been moved to boys
SQL> cm move -conn claire girls
Connection claire has been moved to girls
SQL> cm move -conn marie girls
Connection marie has been moved to girls
SQL> cm move -conn sophie girls
Connection sophie has been moved to girls
SQL> cm list
.
├── boys
│   ├── dick
│   ├── harry
│   └── tom
└── girls
    ├── claire
    ├── marie
    └── sophie
SQL>    
```

We can also ```list``` the contents of a specific folder.
```sql
SQL> cm list -folder boys
boys
├── dick
├── harry
└── tom
SQL>
```


It's worth highlighting that the folder structure is purely for organisation of connections, a connection cannot be referenced using the full path syntax, only with their names. This also means that the connection name must be unique, you cannot have the same connection name in different folders.
```sql
SQL> conn -name boys\dick
Unknown connection boys\dick
SQL> conn -save dick -savepwd dick/dick@localhost:1521/orclpdb1
A connection named dick already exists
SQL>
```

Subfolders are also possible.

```sql
SQL> cm add -folder england
Folder england has been added
SQL> cm move -folder boys england
Folder boys has been moved to england
SQL> cm add -folder france
Folder france has been added
SQL> cm move -folder girls france
Folder girls has been moved to france
SQL> cm list
.
├── england
│   └── boys
│       ├── dick
│       ├── harry
│       └── tom
├── france
│   └── girls
│       ├── claire
│       ├── marie
│       └── sophie
SQL>         
```

If Claire moves to England we can fix that as shown below.  Note as mentioned previously Claire must be referenced by connection name, you cannot use the full path.
```sql
SQL> cm add -folder england/girls
Folder england/girls has been added
SQL> cm move -conn france/girls/claire england/girls
Could not find the specified connection: france/girls/claire
SQL> cm move -conn claire england/girls
Connection claire has been moved to england/girls
```

If all the girls move to England, we are left with some empty folders.  We can either fix that by deleting the parent folder with the -force flag, or by deleting the subfolder, and then the parent.
```sql
SQL> cm move -conn marie england/girls
Connection marie has been moved to england/girls
SQL> cm move -conn sophie england/girls
Connection sophie has been moved to england/girls
SQL> cm list
.
├── england
│   ├── boys
│   │   ├── dick
│   │   ├── harry
│   │   └── tom
│   └── girls
│       ├── claire
│       ├── marie
│       └── sophie
└── france
    └── girls
SQL> cm delete -folder france
The Folder you are trying to delete is not Empty. If you are sure, please retry the command with -force flag
SQL> cm delete -folder france/girls
Folder france/girls has been deleted
SQL> cm delete -folder france
Folder france has been deleted
SQL> cm list
.
└── england
    ├── boys
    │   ├── dick
    │   ├── harry
    │   └── tom
    └── girls
        ├── claire
        ├── marie
        └── sophie   
```