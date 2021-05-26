# Introduction

Masonite ORM was built for the [Masonite Web Framework](https://www.github.com/masoniteframework/masonite) **but is built to work in any Python project**. It is heavily inspired by the Orator Python ORM and is designed to be a drop in replacement for Orator. Orator was inspired by Laravel's Eloquent ORM so if you are coming from a framework like Laravel you should see plenty of similiarities between this project and Eloquent.

Masonite ORM is a beatiful implementation that includes models, migrations, a query builder, seeds, command scaffolding, query scopes, eager loading, model relationships and many more features.

Masonite ORM currently supports MySQL, Maria, Postgres and SQLite 3+ databases.

## Commands

It is important that commands here are based on the standalone version of the package. If you are using Masonite, you can substitute `masonite-orm` with `craft`. If you are not using Masonite, you will just need to use `masonite-orm`:

Standalone:

```
$ masonite-orm model User
```

With Masonite:

```
$ python craft model User
```

Either one of these commands will work with Masonite but to standardize the commands you should use `craft`.
