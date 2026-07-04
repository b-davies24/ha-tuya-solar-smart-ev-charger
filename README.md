# Making a Tuya-connnected EVSE somewhat solar-smart

## Context

Often when you have both an EVSE and solar inverter from the same company, you can use smart features that can dynamically adjust the rate of charge to 'soak' excess solar that would otherwise be exported to the grid. This effectively means you can charge your car 'for free' (i.e. only using your excess solar). There are chargers available that can do this with near-universal compatibility - they tend to use CT clamps around cables in the meterbox, but there was one key limitation of this (arguably more reliable) approach: **a single-phase EV charging on a three-phase capable EVSE**

When being billed, we (at least in Victoria in a normal house) are billed on *net power*, rather than *per phase*. This means that theoretically, you could be importing 10kw on L1, and as long as you export 5kw on both L2 and L3 (or 7kw and 3kw, 6kw and 4kw, etc.), your *net* power would be **zero**. This is great for if you have a car that can only charge on a single phase and a 3-phase house; you can still have net-zero grid import, even if you are importing on one single phase! This is where the spanner is thrown into the works: does XYZ's EVSE with CT clamps do this net power thing, or does it deal with power *only* per phase? If it's the latter, that means that your single-phase car would be getting only a third of the power it could be, and thus take 3 times as long to charge - hardly convenient.

Bunnings has/had a fairly well-priced Tuya-compatible (it's technically 'GRiD Connect', but it can be added to Tuya) 3-phase EVSE. This seemed the 'safer' bet, as it could be quickly integrated to Home Assistant locally (using Tuya Local from HACS: https://github.com/make-all/tuya-local), and should :tm: be able to be programmed to dynamically adjust based on **net power**.

So with that all out of the way, here were the goals:
 - Be able to dynamically adjust the charge rate of the EVSE with relation to net power to/from the grid
 - Allow a single-phase car to charge at a higher rate than theoretically would be allowed by other solutions - i.e. allow importing one phase from the grid as long as the net grid import was zero
 - Be (mostly) intuitive
 - Have the ability to self-stop during peak tarrifs (between a certain set of hours)
