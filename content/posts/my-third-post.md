---
title: "Hacking My Way Around IPv6 Dynamic PD"
date: 2023-11-18T11:00:17+05:30
draft: true
---

Before we get into the problem, let's talk about IPv6 and Dynamic PD. Imagine the internet as a giant city. In this city, every house, every store, every office has a unique address. This address is like an IP address, which is how devices on the internet are identified and can communicate with each other.

In the past, the city used IPv4 addresses. These addresses were like street addresses, and they worked well for a while. Bust as the city grew, there weren't enough street addresses to go around. This made it difficult for new devices to get connected to the internet.

## IPv6: A Next-Level Addressing Revolution

IPv6 is like a new addressing system for the city on steroids. It's like using a combination of street addresses, postal codes, and GPS coordinates to make sure that every every device has a unique and identifiable address. 

While IPv4 had a limited number of 4.3 billion addresses (4,294,967,296, to be exact), IPv6 is practically limitless. It's like expanding from a quaint village to an intergalactic empire, ensuring every toaster, smartphone, and interstellar spaceship gets its unique code.

## Dynamic  PD: The Shape-Shifting Sidekick

Dynamic PD is like the magical real estate agent of the digial world. When you move to a new house, you don't have to get a new street address. Instead, you just tell the postal service your new address, and they will start delivering your mail there. Dynamic PD works the same way. It allows devices to get new IP addresses without having to go through a lot of hassle.

Now let's break down how IPv6 works from both the perspective of an home nework and of an Internet Service Provider:

### IPv6 from the Home Network Side:

1. Router Configuration:

    - The home router receives a dynamic IPv6 prefix from the ISP through the Dynamic Prefix Delegation process.
    
    - The router is configured to handle the dynamic assignment of IPv6 addresses to devices within home network.

2. Device Configuration:

    - Devices within the home network, such as computers, smartphones, and IoT appliances, uses Stateless Address Autoconfiguration (SLAAC) or DHCPv6 to obtain IPv6 addresses.

    - SLAAC allows devices to generate their IPv6 addresses based on the router's prefix.

3. Local Network Communication:

    - Devices on the local network can communicate with each other using their IPv6 addresses without needing to go through Network Address Translation (NAT), as is common in IPv4 networks.

### IPv6 from the ISP Side:

1. Address Allocation:

    - The ISP is assigned blocks of IPv6 addresses by a regional Internet registry (RIR).
    - These addresses are then distributed to customers based on their needs and the types of service plan they have.

2. Prefix Delegation (PD):

    - ISPs use Prefix Delegation (PD) to assign prefixes to customer premises equipment (CPE) or home routers.

## Why Is Dynamic PD BAD?

This is something that a lot of people disagree in the Indian ISP scene. They advertise that a constantly changing delegated IPv6 prefix is normal, standard, and secure. I call BS on it. 

Here's what happens with my ISP (Bharat Sanchar Nigam Ltd. - BSNL) [AS9829]; they reset PPP every 24 hours and the delegated IPv6 prefix change with it. For someone who is a novice, that sounds normal right?, but it isn't. The old IPv6 addresses are still valid and your devices like laptops, phones, TVs, connected to your router will prefer those old v6 address and use it as default while SLAAC gives fresh addresses. Now your devices are using the deprecated addresses to send traffic out and what happens to the response? You guessed it right, they never reach back to your devices. The result is broken IPv6 connectivity.

You can read more about this in here:

Is your ISP constantly changing the delegated IPv6 prefix on your CPE/router?[https://www.6connect.com/blog/is-your-isp-constantly-changing-the-delegated-ipv6-prefix-on-your-cpe-router/]

ISPs: Simplifying customer IPv6 addressing (Part 1)[https://blog.apnic.net/2017/07/07/isps-simplifying-customer-ipv6-addressing-part-1/]

Most users will never notice this as the applications you use will fallback to IPv4. This ignorance is what keeps ISPs in India to continue with half-baked IPv6 configurations.

## What Action Did I Take?

Well, like any normal user I contacted my ISP and I wasn't surprised when these "experts" like my friend Daryll Swer likes to call them, couldn't understand the issue here. I tried explaining to them why Dynamic PD is bad and how an ISP as big as BSNL should adhere to standards and best practices. It seemed like we both were talking different languages. 

I put a halt on pursuing my ISP to fix it and try to find quick-fixes ("jugaad") to stop IPv6 from breaking every 24-hours. Disabling IPv6 was never an option as I had to fight for months to get it enabled, and that's worth an entire blog post of itself.

1. Jugaad No.1 : Using DHCPv6 IA_NA

    DHCPv6 is the IPv6 version of DHCP (well, sort of). Within DHCPv6, the Information Request (IA_NA) option serves a specific purpose in the process of obtaining IPv6 addresses for devices.

    - IA_NA (Identity Association for Non-Temporary Addresses):

        IA_NA is a key option within DHCPv6 used for obtaining non-temporary(i.e., stable) IPv6 addresses for a device. While SLAAC is stateless and provide temporary addresses, DHCPv6 IA_NA offers a stateful mechanism for obtaining stable addresses. Unlike temporary addresses generated by SLAAC, DHCPv6 IA_NA does not keep and use stale IPv6 addresses after ISP dynamically change IPv6 prefix.
    
    This solution comes with a caveat; **Android devices don't support DHCPv6.**

2. Juaggad No.2 : Changing RA Timers

    The reason why I call changing RA Timers a "Juagaad" is because it works but I am probably breaking a bunch of RFCs by doing it. 
    
    My pfSense firewall comes with these default values:
    
        -   Valid Lifetime : 86400 seconds
        -   Preferred Lifetime : 14400 seconds
        -   Minimum RA Interval : 200 seconds
        -   Maximum RA Interval : 600 seconds    
    
    
    The current setup that works for me now is:

        -   Valid Lifetime : 7200 seconds
        -   Preferred Lifetime : 60 seconds
        -   Minimum RA Interval : 3 seconds
        -   Maximum RA Interval : 5 seconds

    This for some reason negates the issue with IPv6 Dynamic PD. My ISP resets PPPoE every 24 hours and now my devices use new prefixes for IPv6 connectivity. The downside of this approach is that it creates a lot of noise in the network. Sending RA every 3-5 seconds will definitely have some toll on battery life of devices as well.

    I do not advice anyone to follow these steps instead pursue your ISPs to do IPv6 static PD, as that is the correct and only proper solution for this.


I have been pursuing my ISP (AS9829) to swtich to follow BCOP-690 and give out /56 static PD for residential users. Being a government funded ISP, I do not see it happening anytime soon as most companies follow policy based engineering instead engineering based policies (which I will talk about in another blog post).

Right now, there are no major telcos or ISPs in my knowledge that do static PD in India. Even if it is going to happen, I believe Jio (AS55836) is going to be the first.

    

    

    









