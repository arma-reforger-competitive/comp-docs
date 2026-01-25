# ARC Competitive AAS v1 Ruleset

**Mode:** Advance and Secure (AAS)  
Infantry and vehicles, ticket-based, linear objectives

---

## 1. Purpose & Scope

This document defines the **official rules** for the competitive AAS mode in Arma Reforger.  

It establishes:

- Match structure and flow  
- Objective and ticket mechanics  
- Spawn and vehicle rules  
- Win conditions  
- Restrictions for fair, balanced competitive play  

This ruleset is suitable for **tournament play**.

---

## 2. Match Overview

- **Teams:** 2 (equal size)  
- **Objectives:** 5–7 linear capture points (flags)  
- **Tickets:** 400 per team (adjustable)  
- **Focus:** Infantry and vehicle combat, tactical coordination  

---

## 3. Flag / Objective Rules

- Flags follow **linear progression** (cannot be skipped)  
- States: Locked → Neutral → Capturing → Secured  
- Capture speed scales with players in the zone  
- Capture time: **90 seconds base**  
- Flags can be **occupied by infantry or vehicles**, but larger vehicles **do not accelerate capture**  

---

## 4. Ticket System

| Event                  | Ticket Change       |
|------------------------|------------------|
| Player death           | -1               |
| Vehicle destroyed      | -5 to -15*       |
| Flag lost              | -20 (losing team) |
| Flag defended           | +30 (holding team) |
| Ticket bleed           | -1 every 30s when behind |

\*Vehicle ticket cost varies by type (light, heavy, armored).  

- Bleed starts when a team controls **fewer flags than the enemy**  
- Final flag imposes **heavy bleed** (-3 per 30s)

---

## 5. Vehicle Rules

- Vehicles are **limited per team** (configurable per layer)
- No vehicle stacking on a single flag to prevent imbalance  
- Vehicle respawn **resets on destruction or ticket loss**  

---

## 6. Spawn Rules

Respawns are strictly limited to **designated team-controlled locations** to maintain competitive fairness.  

### Allowed Spawn Points

1. **Main Base** – Fixed map location; all team members may respawn here.  
2. **Small Radio (Forward Outpost)** – Deployable by a squad's **Radio Operator**; only that squad may respawn here.  
3. **MCU (Mobile Command Unit)** – Truck-mounted mobile spawn; all team members may respawn while it is deployed and powered.  
4. **HUB** – Stationary spawn point deployed by the **Construction Truck**; all team members may respawn while it remains intact.

### Rules

- Players **cannot respawn** in:
  - Contested flags (currently being captured by the enemy)  
  - Locked flags (flags not yet active in linear progression)  

- **Small Radio Spawn**:
  - Only active while deployed and powered by the Radio Operator  
  - Only the deploying squad may use it  
  - Respawn disabled if the Small Radio is destroyed or out of power  

- **MCU Spawn**:
  - Mobile and can be relocated  
  - All team members may use it while deployed  

- **HUB Spawn**:
  - Stationary, must be deployed at a suitable location  
  - All team members may use it while intact  

- Spawn timers are **server-controlled** to prevent abuse or instant redeployment.  
- Vehicles **respawn only at Main Base, MCU, or HUB**; never in contested or locked flags.  

> ⚠️ These rules maintain linear objective progression, prevent spawn camping, and keep ticket penalties meaningful.

---

## 7. Match Flow

1. Warm-up (60 seconds)  
2. Live match (up to 75 minutes)  
3. Overtime triggered if final flag contested when tickets reach 0  
4. Match ends when:
   - Enemy tickets reach 0  
   - Final flag secured  
   - Time expires (team with ticket advantage wins)  

---

## 8. Win Conditions

- Tickets = 0 → enemy loses  
- Final flag secured → controlling team wins  
- Time expires → team with more tickets wins  
- Draw only if tickets equal and no active capture  

---

## 9. Restrictions

- Class / vehicle limits enforced per squad and team
- No unlimited heavy armor or artillery  
- No loadout abuse  
- Spawn camping mitigated by timers and location rules  

---

## 10. Versioning & Authority

- File: `aas-v1.md`  
- Semantic versioning recommended: **v1.0.0, v1.1.0, etc.**  
- This ruleset is the **official Phase 1+ competitive standard**  
- Servers running this mode must follow it strictly
