---
title: 3rd Gen AMD Ryzen build for FreeNAS
excerpt: >
    Check our build based on the AMD Ryzen 7 3700X processor and support for
    error correction code (ECC) memory. We spent $683.96 + tax and shipping for
    the key components. We also have guidelines and suggestions for the
    additional components you'll need to complete your build.
categories:
tags:
  - amd
  - ryzen
  - freenas
  - ecc
toc: true
---

Key components:

* Processor: AMD Ryzen 7 3700X
* Motherboard: ASRock X570 Pro4
* Memory: Kingston KSM26ED8/16ME

The following sections explain the rationale for the key components.

## AMD Ryzen 7 3700X processor

The 3700X processor has the right balance for us:

* Price
* Processing power
* Power consumption

Its eight cores and base clock speed of 3.6GHz are enough for most small offices
workloads, such as hosting a Nextcloud instance or a [Git server][4]. Rating at
65W thermal design power (TDP), it's a better proposal in regards to power
consumption compared to its 3800X and 3900X siblings, which both rate at 105W
TDP.

**Note**: TDP is not an indicator of power consumption; it's an indicator of
heat output. However, lower TDP values amount to lower power consumption in most
cases.
{: .notice }

The AMD Ryzen 7 3700X sells for [$329.99 on Newegg][0]{: target="external"} at
the time of writing this article. A decent cooler is included for this price, so
you don't need to buy another one.

## ASRock X570 Pro4 motherboard

The ASRock X570 motherboards are not usually recommended for gaming purposes due
to its voltage regulator module (VRM) being less than optimal for higher voltage
CPUs or overclocking.

However, we are not using the motherboard in a gaming build. We are using a CPU
with lower TDP and we are not planning on overclocking. So we decided to take a
closer look. We found the following benefits for our scenarios:

* Support for ECC memory
* LAN Compatibility
* Plenty of SATA connections
* Price

The X570 Pro4 offers support for ECC memory, which you should at least consider
for your FreeNAS server. It also includes an Intel Gigabit LAN interface, which
is preferred over other interfaces because FreeNAS has better driver support for
it. This motherboard includes eight SATA connections, which gives plenty of
space for hard drives without the need of expansion cards.

It sells for [$169.99 on Newegg][1]{: target="external"} at the time of this
writing.

You can also check the ASRock X570 Phantom Gaming 4, which sells for [$154.99
regularly on Newegg][3]{: target="external"}, but it's on sale for $134.99 at
the time of this writing. The only reason why we didn't get this motherboard is
because we couldn't get it on time for our needs.
{: .notice }

## Kingston KSM26ED8/16ME memory

At the time of writing this article, the 16GB Kingston KSM26ED8/16ME is the
largest DDR4 memory module supported by the ASRock X570 motherboards that also
features ECC support.

The KSM26ED8/16ME sells for [$91.99 on Newegg][2]{: target="external"} for each
memory module at the time of this writing. You should get at least two to take
advantage of the dual memory channels and have a decent amount of RAM for your
workloads.

## Additional components

You'll need the following additional components to complete your build. We offer
some guidelines and recommendations, but you should take your own considerations
and make your own calculations for your needs.

* Video card: You will need a video card to configure the server. However, you
  don't need a powerful GPU to get the job done. There are many cards [under $15
  on Newegg][6]{: target="external"} that work for this purpose.
* Computer case: We prefer cases that have enough room for multiple hard drives.
  The [Silverstone GD08][5]{: target="external"} includes a removable hard drive
  bracket that makes working with hard drives easier. It also includes options
  to install multiple fans to keep the air flowing through the system.
* Power supply unit (PSU): A PSU of 650W that is at least gold-certified should
  be enough for builds similar to the one in this article. However, you should
  make your own calculations and make sure you have an appropriate PSU. A
  semimodular PSU gives enough flexibility to connect different cable lengths to
  reach all the hard drives in your build.
* Hard drives: Some vendors offer drives that are best suited for NAS systems.
  ~~The WD Red series of drives have the right combination of size, features, and
  price for our systems.~~

  **Warning:** There's some controversy regarding the SMR technology used in
  some of the hard drives mentioned in this article. For more information, see
  the [On WD Red NAS Drives][11]{: target="external"} post and some opinions on
  the [Reddit community][12]{: target="external"}.
  {: .notice--warning }

* USB stick: It's recommended to install FreeNAS on a 16 GiB USB drive. Use a
  USB 2.0 drive instead of 3.0 for better compatibility. You can find some good
  options [on Newegg for under $15][8]{: target="external"} a piece.

## No CPU temperatures on FreeNAS 11.2

Our FreeNAS 11.2 U6 installation doesn't report temperatures from the AMD
processor.
It looks like support is planned for [FreeNAS 11.3][9]{: target="external"}. For
more details, check the [relevant issue][10]{: target="external"} on the FreeNAS 
reporting system.



[0]: https://www.newegg.com/amd-ryzen-7-3700x/p/N82E16819113567?Description=AMD%20Ryzen%207%203700X&cm_re=AMD_Ryzen_7_3700X-_-19-113-567-_-Product
[1]: https://www.newegg.com/p/N82E16813157886?Description=ASRock%20X570%20Pro%204%20AM4&cm_re=ASRock_X570_Pro_4_AM4-_-13-157-886-_-Product
[2]: https://www.newegg.com/kingston-16gb-288-pin-ddr4-sdram/p/1B4-00M4-000U6?Description=Kingston%20KSM26ED8%2f16ME&cm_re=Kingston_KSM26ED8%2f16ME-_-1B4-00M4-000U6-_-Product
[3]: https://www.newegg.com/p/N82E16813157884?Description=ASRock%20X570&cm_re=ASRock_X570-_-13-157-884-_-Product
[4]: /git-server-freenas/
[5]: https://silverstonetek.com/product.php?pid=331&area=en
[6]: https://www.newegg.com/Desktop-Graphics-Cards/SubCategory/ID-48?name=Desktop%2DGraphics%2DCards&Order=PRICE
[7]: https://www.westerndigital.com/products/internal-drives/wd-red-hdd
[8]: https://www.newegg.com/p/pl?d=usb+drive&N=100008022%20601113576%20600082308
[9]: https://redmine.ixsystems.com/versions/859
[10]: https://redmine.ixsystems.com/issues/64077
[11]: https://blog.westerndigital.com/wd-red-nas-drives/
[12]: https://www.reddit.com/r/DataHoarder/comments/g5fs7j/on_wd_red_nas_drives_western_digital_corporate/
