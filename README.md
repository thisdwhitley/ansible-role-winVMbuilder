Ansible Role: winVMbuilder
==========================

Microsoft® provides
[free Windows virtual machines](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/)
to allow customers to test their web browsers, which include Microsoft Edge and
Internet Explorer versions 8 to 11. This VM can be used unrestricted for 90
days.

Because I use Linux every day, I already have a built in hypervisor, 
[KVM](https://www.redhat.com/en/topics/virtualization/what-is-KVM).
Unfortunately, that isn't one of the platforms for which Microsoft provides a
VM...

This role will take some steps to get you a working Windows VM on your Linux
system (I am using Fedora).  At a high level, this role does the following:

1. Sets up a repo and installs some required packages on the Ansible host
2. Downloads a prebuilt *VirtualBox* VM from the very generous Microsoft
3. Unpacks, and converts that file to something we can use with KVM
4. Make some changes prior to starting the VM from this image...*I sort of
   consider this cheating because I couldn't get the networking figured out
   during conversion*
5. Fire up the VM, wait for drivers to load, and reboot since it is required

The result is a freshly built and running Windows VM.  I have a lot of larger
ideas for this role and the VM it creates, but this is it so far...but to this
end, the role will add the new VM to an inventory group named `allVMs`.

Important Notes
---------------

* I have only tested this on my Fedora laptop, so ***YMMV***
* This relies a ***lot*** on your KVM/libvirt (which I will use synonymously
  throughout) configuration, make sure that is set up or you'll get really
  confused.  I speak from experience
* Currently this role uses the `default` storage pool and `default` network
  because I felt those were most likely available.  Feel free to modify per your
  specific situation.
* I changed the NIC to use e1000 until I figure out the virtio drivers during
  conversion...
* Some commands require `root` so ensure you have ansible set up to account
  for this
* This is currently developed to be used on a system locally.  In order to use
  it with Tower or AWX will take some refactoring
* The VM boots up and logs in as the `IEUser` automatically, but so that you
  don't have to search the internets, the password for that account is:
  `Passw0rd!`
* This role is far from idempotent at this point.
* The role will add the newly created VM into a group named `allVMs` which
  is cool because subsequent plays can then use `hosts: allVMs` and it will
  act on the VM created in this role.  Bingo bango.
* There are a lot of factors to determine how long this role will take to run.
  But be aware that we do a few conversions and I don't clean up the files until
  the end so make sure you have enough disk space

Requirements
------------

I couldn't escape a few requirements:

* You'll need to have your libvirt environment configured to your liking.  I'm
  working on creating my own preferences in a separate role...to be continued...

Role Variables
--------------

* `VMurl` *(optional)* a specific VM to install, providing the entire URL as
  the `VMurl` variable.  If none is specified, it defaults to:

  <https://az792536.vo.msecnd.net/vms/VMBuild_20180102/VirtualBox/IE11/IE11.Win7.VirtualBox.zip>

*See the below example which might answer any questions*


Example Playbook
----------------

Playbook specifying a particular VM (by providing the URL) and utilizing the
`allVMs` group for a subsequent play (note, in order to use Ansible on a Windows
system, you've got to jump through some other hoops):

```yaml
- hosts: localhost
  connection: local
  roles:
    - role: winVMbuilder
      VMurl: "https://az792536.vo.msecnd.net/vms/VMBuild_20180102/VirtualBox/IE11/IE11.Win81.VirtualBox.zip"

- hosts: allVMs
  roles:
    - role: do_something_to_the_VM_just_created
```

Inclusion
---------

* I'm imagining this role as a part of other roles, such as to set up a demo
  of Ansible.  So I have included a fairly generic `meta/main.yml` file
  which allows for something similar to:

        ansible-galaxy install -p ./roles -r requirements.yml

    with `requirements.yml` containing:

        ---
        # get the winVMbuilder role from github
        - src: https://github.com/thisdwhitley/ansible-role-winVMbuilder.git
          scm: git
          name: winVMbuilder

References
----------

* https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/
* https://docs.fedoraproject.org/en-US/quick-docs/creating-windows-virtual-machines-using-virtio-drivers/
* https://www.redhat.com/en/blog/importing-vms-kvm-virt-v2v

License
-------

Red Hat, the Shadowman logo, Ansible, and Ansible Tower are trademarks or
registered trademarks of Red Hat, Inc. or its subsidiaries in the United
States and other countries.

All other parts of this project are made available under the terms of the [MIT
License](LICENSE).