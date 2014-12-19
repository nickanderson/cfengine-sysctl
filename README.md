# Manage Sysctl Values in /etc/sysctl.conf

## How to configure this policy

First define a knowledge bundle. This is where you will configure variables
that will ultimately be set into /etc/sysctl.conf. These variables could be
populated directly in policy, or they could load their values from external
data files. The more values you need to set the more it might make sense to
push values out to data files. (JSON is most flexible and reccomended for
external data files, but use what works best for your team. There is nothing
wrong with CSV)

### Required Variables

* file[sysctl] string The path to the sysctl configuration file you wish to
  modify
* sysctl[token] string An individual sysctl token setting, for example
  sysctl[kernel.domainname]

### Example Knowldege Bundle

Here is a simple example that includes automatically calculating a value based
on the ammount of memory available.

```cf3
bundle common chevron_sysctl_settings
{
  vars:
    !windows::
      "file[sysctl]" string => "/etc/sysctl.conf";

    linux::
      "calc_shmall" string => eval("$(mon.value_mem_total) * (0.85) * 1024 * 1024 * 1024 / 4096",
                                   "math",
                                   "infix");

      "sysctl[vm.dirty_expire_centisecs]" string => "1000";
      "sysctl[vm.dirty_ratio]" string => "10";
      "sysctl[vm.dirty_background_ratio]" string => "5";
      "sysctl[vm.overcommit_memory]" string => "1";
      "sysctl[vm.swappiness]" string => "0";
      "sysctl[kernel.shmall]" string => "500",
        comment => "Some arbitrary default value for linux machines";


    linux.San_Ramon::
      "sysctl[kernel.shmall]" string => format("%ld", "$(calc_shmall)"),
        comment => "We are fancy in San Ramon and we automatically calculate the proper size for this value.";
}
```

### Activating this policy

To activate the policy simple use a methods type promise as shown in the
example below.

```cf3
  methods:
      "Sysctl" usebundle => manage_sysctl_conf_selective_present(chevron_sysctl_settings);
```
