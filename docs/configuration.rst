Configuration
=========================



Virtual Server Configuration file
------------------------------------------------

There's one central virtual server configuration file which is **~/.infrasim/.node_map/default.yml** (`source code <https://github.com/InfraSIM/infrasim-compute/blob/master/template/infrasim.yml>`_). All adjustable parameters are defined in this file. This is the only file to modify if you want to customize or make adjustment on the virtual server node. While not all supported options are explicitly listed in this file for purpose of simplicity. However there's one example configuration file - **/etc/infrasim.full.yml.example** (`source code <https://github.com/InfraSIM/infrasim-compute/blob/master/etc/infrasim.full.yml.example>`_) - listed all supported parameters and definitions. By referring content in example file, you can `update node configuration <https://github.com/InfraSIM/infrasim-compute/wiki/Manage-node-config>`_ and then restart ``infrasim node`` service and then new properties will take effect.

   Here's steps for this example::

    # Operating against node with configuration update
    sudo infrasim node destroy <node>

    # Edit node configuration
    sudo infrasim config edit <node>
    # or update a yaml file to the node
    sudo infrasim config update <node> <updated_yml>

    # Start with new configuration
    sudo infrasim node start <node>

.. caution:: To load the newly updated configuration, you must destroy runtime instance, then start this node again.

If you want to manage your own configuration and start infrasim instance accordingly, refer to this article: `Manage Node Config <https://github.com/InfraSIM/infrasim-compute/wiki/Manage-node-config>`_

Here's full list of the example configuration file; every single key-value pair is supported to be add/modify in your real-in-use infrasim.yml::

    # This example virtual server configuration file intends to throughout
    # list parameters and properties that infrasim-compute virtual server
    # supports to adjust. In most cases it is fine to use default value
    # for particuar configuration by skipping putting it into infrasim.yml
    # configuration file. For anything item you're interested, it is recommended
    # to look up infomation here first. For example, if you'd like to customize
    # properties of your drive - either serial number or vender - in below there're
    # corresponding item to show how to achieve that.

    # Unique identifier
    name: node-0

    # Node type is mandatory
    # Node type of your infrasim compute, this will determine the
    # bmc emulation data and bios binary to use.
    # Supported compute node names:
    #  quanta_d51
    #  quanta_t41
    #  dell_c6320
    #  dell_r630
    #  dell_r730
    #  dell_r730xd
    #  s2600kp - Rinjin KP
    #  s2600tp - Rinjin TP
    #  s2600wtt - Node of Hydra, Python
    type: quanta_d51

    compute:
        boot:
            # n - Network (PXE);
            # c - hard disk;
            # d - cdrom;
            boot_order: ncd
            menu: on
            splash: <path/to/your splash picture>
            splash-time: 30000
        kvm_enabled: true
        numa_control: true
        # extra_option can be used to extend qemu command which infrasim did not support yet.
        # an example: -msg timestamp[=on|off]
            # change the format of messages
            # on|off controls leading timestamps (default:on)
        extra_option: -msg timestamp=on
        kernel: /home/infrasim/vmlinuz-3.16.0-25-generic
        initrd: /home/infrasim/initrd.img-3.16.0-25-generic
        cmdline: API_CB=192.168.1.1:8000 BASEFS=base.trusty.3.16.0-25-generic.squashfs.img OVERLAYFS=discovery.overlay.cpio.gz BOOTIF=52-54-BF-11-22-33
        cpu:
            model: host
            features: +vmx
            quantities: 8
        memory:
            size: 4096
        # Currently the PCI bridge is only designed for megasas storage controller
        # When you create multiple megasas controller, the controllers will be assigned
        # a different pci bus number
        pci_bridge_topology:
            -
                device: i82801b11-bridge
                addr: 0x1e.0x0
                multifunction: on
                    -
                        device: pci-bridge
                        chassis_nr: 0x1
                        msi: false
                        addr: 0x1
        storage_backend:
            -
                type: ahci
                max_drive_per_controller: 6
                drives:
                    -
                        model: SATADOM
                        serial: HUSMM142
                        bootindex: 1
                        # To boot esxi, please set ignore_msrs to Y
                        # sudo -i
                        # echo 1 > /sys/module/kvm/parameters/ignore_msrs
                        # cat /sys/module/kvm/parameters/ignore_msrs
                        file: chassis/node1/esxi6u2-1.qcow2
                    -
                        vendor: Hitachi
                        model: HUSMM0SSD
                        serial: 0SV3XMUA
                        # To set rotation to 1 (SSD), need some customization
                        # on qemu
                        # rotation: 1
                        # Use RAM-disk to accelerate IO
                        file: /dev/ram0
                    -
                        vendor: Samsung
                        model: SM162521
                        serial: S0351X2B
                        # Create your disk image first
                        # e.g. qemu-img create -f qcow2 sda.img 2G
                        file: chassis/node1/sda.img
                        page_file: chassis/node1/samsung1.bin
                    -
                        vendor: Samsung
                        model: SM162521
                        serial: S0351X3B
                        file: chassis/node1/sdb.img
                        page_file: chassis/node1/samsung2.bin
                    -
                        vendor: Samsung
                        model: SM162521
                        serial: S0451X2B
                        file: chassis/node1/sdc.img
                        page_file: chassis/node1/samsung3.bin
            -
                type: megasas-gen2
                use_jbod: true
                use_msi: true
                max_cmds: 1024
                max_sge: 128
                max_drive_per_controller: 1
                drives:
                    -
                        vendor: Hitachi
                        product: HUSMM168XXXXX
                        serial: SN0500010351XXX
                        rotation: 1
                        slot_number: 0
                        wwn: 0x50000ccaxxxxxxxx
                        file: <path/to/your disk file>
                        page_file: <path/to/your page bin file>

        networks:
            -
                network_mode: bridge
                # Bridge need to be prepared beforehand with brctl
                network_name: br0
                device: vmxnet3
                mac: 00:60:16:9e:a8:e9
            -
                network_mode: nat
                network_name: ens160
                device: e1000
        ipmi:
            interface: bt
            chardev:
                backend: socket
                host: 127.0.0.1
                reconnect: 10
            ioport: 0xca8
            irq: 10
        smbios: chassis/node1/quanta_d51_smbios.bin
        monitor:
            mode: readline
            chardev:
                backend: socket
                server: true
                wait: false
                host: 127.0.0.1
                port: 2345
        # set vnc display <X>
        vnc_display: 1
        # Set cdrom ISO file for OS installation
        cdrom: /dev/sr0
    bmc:
        interface: br0
        username: admin
        password: admin
        address: <ip address>
        channel: 1
        lancontrol: <path/to/lan control script>
        chassiscontrol: <path/to/chassis control script>
        startcmd: <cmd to be excuted>
        startnow: true
        poweroff_wait: 5
        kill_wait: 5
        historyfru: 20
        config_file: <path/to/your config file>
        emu_file: chassis/node1/quanta_d51.emu
        ipmi_over_lan_port: 623

    # racadm is a segment of attributes defined only for dell server
    racadm:
        # Network to start racadm service
        interface: br0
        port: 10022
        # Credential to access
        username: admin
        password: admin
        # Temporary data provider
        data: /home/infrasim/racadm_data

    # SSH to this port to visit ipmi-console
    ipmi_console_ssh: 9300

    # Renamed from telnet_listen_port to ipmi_console_port, extracted from bmc
    # ipmi-console talk with vBMC via this port
    ipmi_console_port: 9000

    # Used by ipmi_sim and qemu
    bmc_connection_port: 9100

    # Socket file to bridge socat and qemu
    serial_socket: /tmp/serial

Up to infrasim-compute commit `a02417c3 <https://github.com/InfraSIM/infrasim-compute/commit/a02417c37f6b6fb266244e77e992f66938c73f8d>`_

.. _yamlName:

- **name**

    This attribute defines nodes name, which is a unique identifier for infrasim-compute instances on the same platform.
    More specifically, it is used as `workspace <https://github.com/InfraSIM/infrasim-compute/wiki/Compute-Node-Workspace>`_ folder name.

    **NOT Mandatory**

    **Default**: "node-0"

    **Legal Value**: String

.. _yamlType:

- **type**

    This attribute defines supported nodes type in InfraSIM. With this attribute, infrasim-compute will set BMC emulation data for ``ipmi_sim`` and BIOS binary for ``qemu`` accordingly, you can get corresponding .emu and .bin in ``/usr/local/etc/infrasim/`` by default.

    **Mandatory**

    **Legal Values**:

        - "quanta_d51"
        - "quanta_t41"
        - "dell_c6320"
        - "dell_r630"
        - "dell_r730"
        - "dell_r730xd"
        - "s2600kp", for Rinjin KP
        - "s2600tp", for Rinjin TP
        - "s2600wtt", for Hydra, Python

.. _yamlCompute:

- **compute**

    This block defines all attributes used by `QEMU <http://wiki.qemu.org/Main_Page>`_.
    They will finally be translated to one or more ``qemu`` command options.
    The module ``infrasim.model.CCompute`` is handling this translation.
    This is much like a definition for `libvert <https://libvirt.org/>`_, but we may want it to be lite, and compatible with some customized qemu feature in InfraSIM.

.. _yamlComputeBoot:

- **compute:boot**

    This group of attributes set qemu boot characteristics. See ``-boot`` in `qemu-doc <http://wiki.qemu.org/download/qemu-doc.html>`_.

.. _yamlComputeBootorder:

- **compute:boot:boot_order**

    This attribute defines boot order for ``qemu``. Will be translated to ``-boot {boot_order}``.

    **Not Mandatory**

    **Default**: "ncd", means in a order of pxe > disk > cdrom.

    **Legal Value**: See ``-boot`` in `qemu-doc <http://wiki.qemu.org/download/qemu-doc.html>`_.

.. _yamlComputeMenu:

- **compute:boot:menu**

    This attribute can enable interactive boot menus/prompts via ``menu=on`` **as far as firmware/BIOS supports them**.
    If ``menu=on`` is set and the firmware/BIOS supports boot menus, the interactive boot menu will be shown when press the shortcuts according to the hint message at boot time.
    `Here <https://bintray.com/infrasim/generic/download_file?file_path=pool%2Fmain%2FS%2FSeabios%2Finfrasim-seabios_1.1-99ubuntu16.04_amd64.bin>`_ is a bios file which supports interactive boot menus.

    Here is a command line to check whether the bios can support menu or not::

        # boot with an interactive boot menu with 20-second splash time and the bios file "bios.bin"
        qemu-system-x86_64 -boot menu=on,splash-time=20000 -bios bios.bin

    Perform ``infrasim init``, then `this <https://bintray.com/infrasim/generic/Seabios>`_ bios file will be downloaded and saved in ``/usr/local/share/qemu/bios-256k.bin`` as InfraSIM default bios file.

    **Not Mandatory**

    **Default**: None, means non-interactive boot, and there will be no ``menu=on`` or ``menu=off`` option.

    **Legal Value**: ``on`` or ``off``.

.. _yamlComputeSplash:

- **compute:boot:splash**

    This attribute defines the splash picture path. This picture will be passed to bios, enabling user to show it as logo. This splash file could be a jpeg file or a BMP file in 24 BPP format(true color). The resolution should be supported by the SVGA mode, so the recommended is 320x240, 640x480, 800x640.

    **Not Mandatory**

    **Default**: None.

    **Legal Value**: a valid file path, absolute or relative.

.. _yamlComputeSplashtime:

- **compute:boot:splash-time**

    This attribute defines the splash time.

    **Not Mandatory**

    **Default**: None, means splash time is 0.

    **Legal Value**: positive integer. 30000 means 30 seconds.

.. _yamlComputeKvmenabled:

- **compute:kvm_enabled**

    This attribute enable `kvm <http://wiki.qemu.org/Features/KVM>`_ when you announce it as True and your system supports kvm. It will be translated to ``--enable-kvm``. You can check if your system supports kvm by check if ``/dev/kvm`` exists.

    **Not Mandatory**

    **Default**: Depends on if ``/dev/kvm`` exists.

    **Boolean Table**

    +------------+-------------+--------------+
    |kvm_enabled |/dev/kvm     |--enable-kvm  |
    +============+=============+==============+
    |true        |yes          |yes           |
    +------------+-------------+--------------+
    |true        |no           |no            |
    +------------+-------------+--------------+
    |false       |yes          |no            |
    +------------+-------------+--------------+
    |false       |no           |no            |
    +------------+-------------+--------------+
    |not define  |yes          |yes           |
    +------------+-------------+--------------+
    |not define  |no           |no            |
    +------------+-------------+--------------+

.. _yamlComputeNumacontrol:

- **compute:numa_control**

    This attribute enable `NUMA <https://en.wikipedia.org/wiki/Non-uniform_memory_access>`_ to improve InfraSIM performance by binding to certain physical cpu.
    If you have installed ``numactl`` and set this attribute to True, you will run qemu in a way like ``numactl --physcpubind={cpu_list} --localalloc``.

    **Not Mandatory**

    **Default**: Disabled

.. _yamlComputeKernel:

- **compute:kernel**

    This attribute specifies the binary kernel file path. It will be used by qemu to install.

    **Not Mandatory**

    **Default**: None.

.. _yamlComputeInitrd:

- **compute:initrd**

    This attribute specifies the initial ram disk path. This INITRD image can be used to provide a place for qemu to install kernel. See ``-initrd file`` in `qemu-doc <http://wiki.qemu.org/download/qemu-doc.html>`_.

    **Mandatory**: depends on if ``kernel`` is given.

    **Default**: None.


.. _yamlComputeCmdline:

- **compute:cmdline**

    This attribute will be appended to qemu in string as part of the option ``--append {cmdline}``.
    See ``--append`` in `qemu-doc <http://wiki.qemu.org/download/qemu-doc.html>`_.
    It will be then used by qemu as kernel parameters.
    You can view your O/S's kernel parameters by ``cat /proc/cmdline``.

    **Not Mandatory**

    **Default**: None, there will be no ``--append`` option.

.. _yamlComputeCpu:

- **compute:cpu**

    This group of attributes set qemu cpu characteristics. The module ``infrasim.model.CCPU`` is handling the information.

.. _yamlComputeCpuModel:

- **compute:cpu:model**

    This attribute sets qemu cpu model.

    **Not Mandatory**

    **Default**: "host"

    **Legal Values**: See ``-cpu model`` in `qemu-doc <http://wiki.qemu.org/download/qemu-doc.html>`_.

.. _yamlComputeCpuFeatures:

- **compute:cpu:features**

    This attribute adds or removes cpu flags according to your customization. It will be translated to ``-cpu Haswell,+vmx`` for example.

    **Not Mandatory**

    **Default**: "+vmx"

    **Legal Values**: See ``-cpu model`` in `qemu-doc <http://wiki.qemu.org/download/qemu-doc.html>`_.

.. _yamlComputeCpuQuantities:

- **compute:cpu:quantities**

    This attribute sets virtual cpu numbers in all. With default socket 2, CCPU calculates core per socket. Default set to 1 thread per cores.
    It will be translated to ``-smp {cpus},sockets={sockets},cores={cores},threads=1`` for example.

    **Not Mandatory**

    **Default**: 2

    **Legal Values**: See ``-smp`` in `qemu-doc <http://wiki.qemu.org/download/qemu-doc.html>`_.

.. _yamlComputeMemory:

- **compute:memory**

    This attribute refers to RAM, which the virtual computer devices use to store information for immediate use.
    The module ``infrasim.model.CMemory`` is handling the information.

.. _yamlComputeMemorySize:

- **compute:memory:size**

    This attribute sets the startup RAM size. The default is 1024MB.

    **Default**: 1024

    **Legal Values**: See ``-m`` in `qemu-doc <http://wiki.qemu.org/download/qemu-doc.html>`_.

.. _yamlComputeStoragebackend:

- **compute:storage_backend**

    This block defines backend storage details. It maintains a list of ``controller`` structures,
    and each controller maintains a list of ``drive`` structures.

.. _yamlComputeStoragebackendController:

- **compute:storage_backend:-**

    Each element of this list defines a storage ``controller``, they have some common attributes.
    The module ``infrasim.model.CBaseStorageController`` is handling the information.
    Developer may inherits this class to define other type of controller and specific controller attributes.

    Common attributes:

    - type

    - max_drive_per_controller

    Specific controllers defined:

    +----------------------+-------------------------------------+--------------+
    |Controller Type       |Module                               |Attributes    |
    +======================+=====================================+==============+
    |megasas.*             |infrasim.model.MegaSASController     |use_jbod      |
    |                      |                                     |sas_address   |
    |                      |                                     |use_msi       |
    |                      |                                     |max_cmds      |
    |                      |                                     |max_sge       |
    +----------------------+-------------------------------------+--------------+
    |lsi.*                 |infrasim.model.LSISASController      |              |
    +----------------------+-------------------------------------+--------------+
    |.\*ahci.*             |infrasim.model.AHCIController        |              |
    +----------------------+-------------------------------------+--------------+


.. _yamlComputeStoragebackendControllerType:

- **compute:storage_backend:-:type**

    Define types of a controller, this makes infrasim-compute model handle other attributes accordingly.

.. _yamlComputeStoragebackendControllerMaxdrivepercontroller:

- **compute:storage_backend:-:max_drive_per_controller**

    This is a protection mechanism that you write too much in ``drives`` list.
    If the actual count of drives exceeds this limitation, infrasim-compute now make more controller, in the same attribute but different PCI bus number, to mount all drives.
    The module ``infrasim.model.CPCITopologyManager`` defines this logic.

.. _yamlComputeStoragebackendControllerDrives:

- **compute:storage_backend:-:controller:drives**

    This attribute defines a list of ``drives`` mounted on the controller.
    Common attributes are managed by ``infrasim.model.CBaseDrive``.
    Developer may inherits this class to define other type of drive and specific attributes.

    Common attributes - device personality options:

    - bootindex

    - serial

    - wwn

    - version

    Common attributes - simulation options:

    - format

    - cache

    - aio

    - size

    - file
    
    - page-file

    Drive type currently depends on the controller it is mounted on:

    +----------------------+-------------------------------------+--------------+
    |Controller Type       |Mounted Drive Type                   |Attributes    |
    +======================+=====================================+==============+
    |LSISASController      |infrasim.model.SCSIDrive             |port_index    |
    |MegaSASController     |                                     |port_wwn      |
    |                      |                                     |channel       |
    |                      |                                     |scsi-id       |
    |                      |                                     |lun           |
    |                      |                                     |slot_number   |
    |                      |                                     |product       |
    |                      |                                     |vendor        |
    |                      |                                     |rotation      |
    +----------------------+-------------------------------------+--------------+
    |AHCIController        |infrasim.model.IDEDrive              |model         |
    +----------------------+-------------------------------------+--------------+

.. _yamlComputeStoragebackendControllerDrivesBootindex:

- **compute:storage_backend:-:controller:drives:-:bootindex**

    Cite from qemu's `bootindex <https://github.com/qemu/qemu/blob/master/docs/bootindex.txt>`_ documentation.

    Block and net devices have bootindex property. This property is used to
    determine the order in which firmware will consider devices for booting
    the guest OS. If the bootindex property is not set for a device, it gets
    lowest boot priority. There is no particular order in which devices with
    unset bootindex property will be considered for booting, but they will
    still be bootable.

    **NOT Mandatory**

    **Legal Value**: integer

    **Example**: Let's assume we have a QEMU machine with two NICs (virtio, e1000) and two disks (IDE, virtio):

        qemu -drive file=disk1.img,if=none,id=disk1
             -device ide-drive,drive=disk1,bootindex=4
             -drive file=disk2.img,if=none,id=disk2
             -device virtio-blk-pci,drive=disk2,bootindex=3
             -netdev type=user,id=net0 -device virtio-net-pci,netdev=net0,bootindex=2
             -netdev type=user,id=net1 -device e1000,netdev=net1,bootindex=1

    Given the command above, firmware should try to boot from the e1000 NIC
    first.  If this fails, it should try the virtio NIC next; if this fails
    too, it should try the virtio disk, and then the IDE disk.

.. _yamlComputeStoragebackendControllerDrivesSerial:

- **compute:storage_backend:-:controller:drives:-:serial**

    Drive's serial number.

    **NOT Mandatory**

.. _yamlComputeStoragebackendControllerDrivesWWN:

- **compute:storage_backend:-:controller:drives:-:wwn**

    Refer to `WWN (wikipedia) <https://en.wikipedia.org/wiki/World_Wide_Name>`_.

    **NOT Mandatory**

.. _yamlComputeStoragebackendControllerDrivesVersion:

- **compute:storage_backend:-:controller:drives:-:version**

.. _yamlComputeStoragebackendControllerDrivesFormat:

- **compute:storage_backend:-:controller:drives:-:format**

    Cite from `QEMU <http://download.qemu-project.org/qemu-doc.html#index-_002dchardev>`_:

    Specify which disk ``format`` will be used rather than detecting the format. Can be used to specifiy format=raw to avoid interpreting an untrusted format header.

    This attribute will be translated to ``-drive format={format}``.

.. _yamlComputeStoragebackendControllerDrivesCache:

- **compute:storage_backend:-:controller:drives:-:cache**

    Cite from `QEMU <http://download.qemu-project.org/qemu-doc.html#index-_002dchardev>`_:

    ``cache`` is "none", "writeback", "unsafe", "directsync" or "writethrough" and controls how the host cache is used to access block data.

    This attribute will be translated to ``-drive cache={cache}``.

.. _yamlComputeStoragebackendControllerDrivesAio:

- **compute:storage_backend:-:controller:drives:-:aio**

    Cite from `QEMU <http://download.qemu-project.org/qemu-doc.html#index-_002dchardev>`_:

    ``aio`` is "threads", or "native" and selects between pthread based disk I/O and native `Linux AIO <http://man7.org/linux/man-pages/man7/aio.7.html>`_.

    This attribute will be translated to ``-drive aio={aio}``.

.. _yamlComputeStoragebackendControllerDrivesFile:

- **compute:storage_backend:-:controller:drives:-:file**

    Cite from `QEMU <http://download.qemu-project.org/qemu-doc.html#index-_002dchardev>`_:

    This option defines which disk image to use with this drive.

    This attribute will be translated to ``-drive file={file}``.

.. _yamlComputeStoragebackendControllerDrivesSize:

- **compute:storage_backend:-:controller:drives:-:page-file**

    This option allows user to specify drive page data, which can provide addtional information for client OS, including mode sense pages and inquiry data pages. The page file is generated by a tool which can fetch data from HW drive or user defined json file.
    
    Command, e.g. ``sudo python gen_page_utility.py -d /dev/sdb -o drive_name.bin``, will create a drive page bin file.
    
    For more details, please refer to `how-to-generate-drive-page-files <http://infrasim.readthedocs.io/en/latest/how_to.html#how-to-generate-drive-page-files>`_.
    
    This attribute will be translated to ``-device page_file={file}``.

- **compute:storage_backend:-:controller:drives:-:size**

    If infrasim-compute application can't detect existing drive file, it will help user create a drive image file.
    A command, e.g. ``qemu-img create -f qcow2 sda.img 10G``, will be called to create such a drive file in `node workspace <https://github.com/InfraSIM/infrasim-compute/wiki/Compute-Node-Workspace>`_.
    This is where ``size`` take effects.


    **Not Mandatory**

    **Default**: 8

    **Legal Values**: integer, in unit of GB

.. _yamlComputeNetworks:

- **compute:networks**

.. _yamlComputeNetworksNetworkmode:

- **compute:networks:-:network_mode**

.. _yamlComputeNetworksNetworkname:

- **compute:networks:-:network_name**

.. _yamlComputeNetworksDevice:

- **compute:networks:-:device**

.. _yamlComputeNetworksMac:

- **compute:networks:-:mac**

.. _yamlComputeIpmi:

- **compute:ipmi**

.. _yamlComputeIpmiInterface:

- **compute:ipmi:interface**

.. _yamlComputeIpmiChardev:

- **compute:ipmi:chardev**

.. _yamlComputeIpmiChardevBackend:

- **compute:ipmi:chardev:backend**

.. _yamlComputeIpmiChardevHost:

- **compute:ipmi:chardev:host**

.. _yamlComputeIpmiChardevReconnect:

- **compute:ipmi:chardev:reconnect**

.. _yamlComputeIpmiIoport:

- **compute:ipmi:ioport**

.. _yamlComputeIpmiIrq:

- **compute:ipmi:Irq**

.. _yamlComputeSmbios:

- **compute:smbios**

.. _yamlComputeMonitor:

- **compute:monitor**

.. _yamlComputeMonitorMode:

- **compute:monitor:mode**

.. _yamlComputeMonitorChardev:

- **compute:monitor:chardev**

.. _yamlComputeMonitorChardevBackend:

- **compute:monitor:chardev:backend**

.. _yamlComputeMonitorChardevServer:

- **compute:monitor:chardev:server**

.. _yamlComputeMonitorChardevWait:

- **compute:monitor:chardev:wait**

.. _yamlComputeMonitorChardevPath:

- **compute:monitor:chardev:path**

.. _yamlComputeVncdisplay:

- **compute:vnc_display**

.. _yamlComputeCdrom:

- **compute:cdrom**

    This attribute specify a media when qemu boot from cdrom. You can promote cdrom boot order by specify ``d`` first in ``compute:boot:boot_order``.

    **Not Mandatory**

    **Legal Values**: path to a image file, or directly use cdrom device, e.g. ``/dev/sr0``

.. _yamlBmc:

- **bmc**

    This block defines attributes used by `OpenIPMI <http://openipmi.sourceforge.net/>`_.
    They will finally be translated to one or more ``ipmi_sim`` command options, or be defined in the configuration file for it.
    The module ``infrasim.model.CBMC`` is handling this translation.

.. _yamlBmcInterface:

- **bmc:interface**

   This attributes defines both:

   - from which network ``ipmi_sim`` will listen IPMI request

   - BMC's network properties printed by ``ipmitool lan print``

   The module ``infrasim.model.CBMC`` takes this attribute and comes out with two variable defined in ipmi_sim `configuration template <https://github.com/InfraSIM/infrasim-compute/blob/master/template/vbmc.conf>`_.

   - ``{{lan_interface}}``, network name for ``ipmitool lan print`` to print, e.g. "eth0", "ens190".

   - ``{{ipmi_listen_range}}``, IP address that ipmi_sim shall listen to and response IPMI command. If you set a valid interface here, an IP address in string will be assigned to this variable, e.g. "192.168.1.1".

   **Not Mandatory**


   **Default**

   - ``{{lan_interface}}``: first network device except ``lo``.

   - ``{{ipmi_listen_range}}``: "::", so that you shall see ``addr :: 623`` in vbmc.conf, it means ipmi_sim listen to IPMI request on all network on port 623


   **Valid Interface**: Use network devices from ``ifconfig``.

   - ``{{lan_interface}}``: the specified network interface.
   - ``{{ipmi_listen_range}}``: IP address of lan_interface("0.0.0.0" if interface has no IP).


   **Invalid Interface**: Network devices that don't exist.

   - ``{{lan_interface}}``: no binding device
   - ``{{ipmi_listen_range}}``: no range setting, which means user could only access ipmi_sim through kcs channel inside qemu OS.


.. _yamlBmcUsername:

- **bmc:username**

.. _yamlBmcPassword:

- **bmc:password**

.. _yamlBmcAddress:

- **bmc:address**

.. _yamlBmcChannel:

- **bmc:channel**

.. _yamlBmcLancontrol:

- **bmc:lancontrol**

.. _yamlBmcChassiscontrol:

- **bmc:chassiscontrol**

.. _yamlBmcStartcmd:

- **bmc:startcmd**

.. _yamlBmcStartnow:

- **bmc:startnow**

.. _yamlBmcPoweroffwait:

- **bmc:poweroff_wait**

.. _yamlBmcHistoryfru:

- **bmc:historyfru**

.. _yamlBmcConfigfile:

- **bmc:config_file**

.. _yamlBmcEmufile:

- **bmc:emu_file**

.. _yamlBmcIpmioverlanport:

- **bmc:ipmi_over_lan_port**

.. _yamlRacadm:

- **racadm**

    This block defines `RACADM <http://en.community.dell.com/techcenter/systems-management/w/wiki/3205.racadm-command-line-interface-for-drac>`_ (Remote Access Controller ADMin) simulation behavior.

.. _yamlRacadmInterface:

- **racadm:interface**

    This attribute defines on which interface RACADM shall listen to. It will then start as a service, listening on the certain IP.

    **Not Mandatory**

    **Default**: if you don't set this attribute, RACADM will start listening on ``0.0.0.0``

    **Legal Values**: a valid interface with IP address

.. _yamlRacadmPort:

- **racadm:port**

    This attribute defines on which port RACADM shall listen to. It works with the :racadm\:interface:`yamlRacadmInterface`.

    **Not Mandatory**

    **Default**: 10022

    **Legal Values**: a valid port that is not being used

.. _yamlRacadmUsername:

- **racadm:username**

    SSH username on RACADM simulation.

    **Default**: admin

.. _yamlRacadmPassword:

- **racadm:password**

    SSH password on RACADM simulation.

    **Default**: admin

.. _yamlRacadmData:

- **racadm:data**

    You need to specify a folder name for this attribute, e.g. ``/home/infrasim/data``.
    In this folder, you need to provide several pure text files.
    Each file maintains response for a certain RACADM command.

    RACADM simulation now is not getting runtime data from BIOS binary or IPMI emulation data,
    but using this temporary implementation to inject data for RACADM simulation.

    Here is a list of supporting data and required text file name (without extension .txt).

    +--------------------------------------+----------------------------------------+
    |RACADM Command                        |Response File Name                      |
    +======================================+========================================+
    |getled                                |getled                                  |
    +--------------------------------------+----------------------------------------+
    |getsysinfo                            |getsysinfo                              |
    +--------------------------------------+----------------------------------------+
    |storage get pdisks –o                 |storage_get_pdisks_o                    |
    +--------------------------------------+----------------------------------------+
    |get BIOS                              |get_bios                                |
    +--------------------------------------+----------------------------------------+
    |get BIOS.MemSettings                  |get_bios_mem_setting                    |
    +--------------------------------------+----------------------------------------+
    |hwinventory                           |hwinventory                             |
    +--------------------------------------+----------------------------------------+
    |hwinventory nic.Integrated.1-1-1      |hwinventory_nic_integrated_1-1-1        |
    +--------------------------------------+----------------------------------------+
    |hwinventory nic.Integrated.1-2-1      |hwinventory_nic_integrated_1-2-1        |
    +--------------------------------------+----------------------------------------+
    |hwinventory nic.Integrated.1-3-1      |hwinventory_nic_integrated_1-3-1        |
    +--------------------------------------+----------------------------------------+
    |hwinventory nic.Integrated.1-4-1      |hwinventory_nic_integrated_1-4-1        |
    +--------------------------------------+----------------------------------------+
    |get IDRAC                             |get_idrac                               |
    +--------------------------------------+----------------------------------------+
    |setled -l 0                           |setled_l_0                              |
    +--------------------------------------+----------------------------------------+
    |get LifeCycleController               |get_life_cycle_controller               |
    +--------------------------------------+----------------------------------------+
    |get LifeCycleController.LCAttributes  |get_life_cycle_controller_lc_attributes |
    +--------------------------------------+----------------------------------------+

.. _yamlIpmiconsolessh:

- **ipmi_console_ssh**

.. _yamlIpmiconsoleport:

- **ipmi_console_port**

.. _yamlBmcconnectionport:

- **bmc_connection_port**

.. _yamlSerialsocket:

- **serial_socket**

    This attribute defines a `unix socket <https://en.wikipedia.org/wiki/Unix_domain_socket>`_ file to forward data.
    More specifically, it bridges ``socat`` and ``qemu`` for InfraSIM to forward input and output stream as a serial port.
    With this attribute designed, you will see ``socat`` starts with option ``unix-listen:<file>``,
    while ``qemu`` starts with a socket chardev ``-chardev socket,path=<file>,id=...``

    **Not Mandatory**

    **Default**: a file named ``.socket`` in `node workspace <https://github.com/InfraSIM/infrasim-compute/wiki/Compute-Node-Workspace>`_

    **Legal Values**: a valid file path, absolute or relative, to create such node

Networking
------------------------------------------------

#. Virtual server NAT or host-only mode, this is default mode implemented in infrasim-compute
    * vCompute is accessible ONLY inside Ubuntu host 
    * Software running in vCompute can access outside network if connecting Ubuntu host NIC with virtual bridge
    * Configuration YAML file can specify which NIC IPMI over LAN traffic flows through

    .. image:: _static/networking_nat.PNG
        :align: center

#. Bridge mode - single
    * Work as virtual switch
    * Connect BMC NIC and NICs in virtual compute together
    * Configuration YAML file controls how many NICs that virtual compute has and specify bridge they connect to

    .. image:: _static/networking_bridge_single.PNG
        :align: center

    .. note:: It requires setting up bridge and connect to NIC of underlying host in advance. 
    
    Here's steps for this example::

            # brctl addr br0
            # brctl addif br0 eth1
            # brctl setfd br0 0
            # brctl sethello < bridge name > 1
            # brctl stp br0 no
            # ifup br0

#. Bridge mode - multiple

    .. image:: _static/networking_bridge_multiple.PNG
        :align: center


.. hide_content::

            Virtual Power Distribution Unit - Robert - Under construction
            ------------------------------------------------

            Current Virtual PDU implementation only supports running entire virutal infrastructure on VMWare ESXi because it only supports functionality of simulating power control chassis through VMWare SDK.

            .. image:: _static/networkwithoutrackhd.png
                :align: center

