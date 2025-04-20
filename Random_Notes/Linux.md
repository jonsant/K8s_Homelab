# Linux

- rp_filter (Reverse path filtering) defaults to 2 in Ubuntu, see `/proc/sys/net/ipv4/conf/default/rp_filter`. Links:  [1](https://bugs.launchpad.net/ubuntu/+source/procps/+bug/1814262), [2](https://www.reddit.com/r/Ubuntu/comments/1eqo0o2/why_is_ubuntu_using_rp_filter_2_as_the_distro/)

  - i changed it to the more secure 'Strict Mode' by running this as root: `echo 1 > /proc/sys/net/ipv4/conf/all/rp_filter`. More info [here](https://support.luminex.be/portal/en/kb/articles/linux-reverse-path-filtering-and-multicast-packet-drops-on-embedded-devices)

- following the [k8s docs](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#prerequisite-ipv4-forwarding-optional), i enabled IPv4 packet forwarding.
