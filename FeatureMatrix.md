# Introduction #

In the world of Unix, and especially in the world of Unix shells, variety abounds. Some shells support some things, others support something else. shUnit2 tries both to work with the common denominator of shell and OS features, and provide as many useful features as possible. Unfortunately, not all features are supported in all combinations, and it is important to know what works where.

# Features Matrix #

## 2.1.6 ##
| **Feature** | **OS** | **Shell** | **Supported** |
|:------------|:-------|:----------|:--------------|
| Line numbers on asserts | Ubuntu 10.04 | bash 4.1 | **yes** |
|  |  | dash 0.5.5.1 | no |