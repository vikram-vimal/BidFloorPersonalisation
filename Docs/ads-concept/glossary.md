# Glossary

Definitions for the terms used across this documentation space.

## Platform & roles

**Ad unit** — A high-level object in the mediation system that defines all properties of an ad. It has an ID referenced in game code to load an ad, an ad type (interstitial, rewarded, banner, MREC, or native), and a platform (Android or iOS).

**Bid floor** — The minimum price of an ad impression the game is willing to accept as a publisher. Bid floors can be set at country level.

**Ad network (DSP, Demand-Side Partner)** — Advertisers and inventory buyers — companies like Google, Meta, AppLovin, Unity, and e-commerce brands — who want their ads shown to the right users.

**Gaming studio (Publisher / SSP, Supply-Side Partner)** — The studio owns the ad inventory: the moments within gameplay where an ad can be shown.

**Mediation platform** (e.g. AppLovin MAX, AdMob) — Runs the auction. Aggregates bids from multiple DSPs and ad networks, runs a unified auction, and delivers the winning ad. Revenue comes from a percentage cut on each successful impression. The goal is a competitive auction that produces the highest possible winning bid.

**Ad placement** — Where in the game the ad appears: which page or screen the user was on when they watched the ad.

## Auction types

### Waterfall auction

A publisher passes inventory from ad network to ad network in descending order of priority until impressions are sold. Historically popular for its simple technical setup.

### First-price auction

What you bid is what you pay. If three advertisers bid $1, $2, and $3, the $3 bidder wins and pays $3. Google migrated to this model in 2021; it is now the industry standard in mobile mediation.

### Second-price auction

The winning bidder pays $0.01 more than the second-highest bid. If bids are $1, $2, and $3, the winner pays $2.01. This discouraged bid manipulation but is largely deprecated in mobile mediation.
