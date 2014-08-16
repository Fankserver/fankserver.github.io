Nice 2 know
===========

Perl
----

CPAN first setup use default configuration
```shell
PERL_MM_USE_DEFAULT=1 cpan
```

CPAN install all prerequisites, enter following in CPAN:
```shell
o conf prerequisites_policy 'follow'
o conf build_requires_install_policy yes
o conf commit
```
