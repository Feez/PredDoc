# DreamPred API Documentation

## Initialization
```lua
function OnLoad()
    if not _G.Prediction then
    	LoadPaidScript(PaidScript.DREAM_PRED)
    end 
end
```
We check if `_G.Prediction` exists before loading DreamPred in order to avoid unnecessary, repeated script loads. The built-in `require` is different in that it automatically avoids this, but `LoadPaidScript` does not.

## Core functions
```lua
Prediction.GetPrediction(unit, spell, source)
```

**Parameters:**

| Name   | Type                    | Description                             |
| ------ | ----------------------- | --------------------------------------- |
| unit   | GameObject              | The target unit                         |
| spell  | table [**`SpellData`**] | The spell data you want to use          |
| source | Vector/D3DXVECTOR3      | The starting position of the spell cast |

**Returns:**

| #   | Type                     | Description          |
| --- | ------------------------ | -------------------- |
| 1   | table [**`predResult`**] | Result of prediction |

---
```lua
Prediction.GetUnitPosition(unit, delay)
```

**Parameters:**

| Name  | Type       | Description        |
| ----- | ---------- | ------------------ |
| unit  | GameObject | The target unit    |
| delay | number     | Delay (in seconds) |

**Returns:**

| #   | Type        | Description                                       |
| --- | ----------- | ------------------------------------------------- |
| 1   | D3DXVECTOR3 | Resulting position from prediction                |
| 2   | number      | Probability of unit being at position after delay |

---
```lua
Prediction.IsDashing(unit, spell, source)
```

**Parameters:**

| Name   | Type                    | Description                             |
| ------ | ----------------------- | --------------------------------------- |
| unit   | GameObject              | The target unit                         |
| spell  | table [**`SpellData`**] | The spell data you want to use          |
| source | Vector/D3DXVECTOR3      | The starting position of the spell cast |

**Returns:**

| #   | Type   | Description                              |
| --- | ------ | ---------------------------------------- |
| 1   | bool   | If target is dashing                     |
| 2   | bool   | If target can be hit                     |
| 3   | Vector | Predicted position of target during dash |

*Note: This is implicitly called in `GetPrediction`, it will return hitChance of 1 if the target is dashing & is hittable.*

---
```lua
Prediction.IsCollision(spell, startPos, endPos, target)
```

**Parameters:**

| Name     | Type                    | Description                                   |
| -------- | ----------------------- | --------------------------------------------- |
| spell    | table [**`SpellData`**] | The spell data you want to use                |
| startPos | Vector/D3DXVECTOR3      | The start position of the spell               |
| endPos   | Vector/D3DXVECTOR3      | The end position of the spell                 |
| target   | GameObject              | Target you want to see if collides with spell |

**Returns:**

| #   | Type | Description                                        |
| --- | ---- | -------------------------------------------------- |
| 1   | bool | If is collision along the trajectory with 'target' |

*Note: This is not the same as what is used with built-in hero/minion collision. This does not include any collision buffers and is not conservative in returning if there is a collision.*

## Helper functions

```lua
Prediction.IsRecalling(unit, delay)              -- works for FoW
Prediction.IsPreparingRecall(unit, delay)
Prediction.IsTeleporting(unit, delay)            -- works for FoW
Prediction.IsImmobile(unit, delay)
Prediction.IsAttacking(unit, delay)
```

**Parameters:**

| Name  | Type       | Description                                                  |
| ----- | ---------- | ------------------------------------------------------------ |
| unit  | GameObject | Unit you want to test condition for                          |
| delay | number     | Number of seconds in future to make sure condition is active |

**Returns:**

| #   | Type | Description                                         |
| --- | ---- | --------------------------------------------------- |
| 1   | bool | If condition will be true after the specified delay |
| 2   | bool | Time left of condition excluding delay              |

---
```lua
Prediction.IsFacing(unit, position, angle)
```

**Parameters:**

| Name     | Type               | Description                                                         |
| -------- | ------------------ | ------------------------------------------------------------------- |
| unit     | GameObject         | Unit you want to see if is facing position                          |
| position | Vector/D3DXVECTOR3 | Position you want to see if 'unit' is facing                        |
| angle    | number             | Maximum angle rotated from 'unit' direction that contains position. |


**Returns:**

| #   | Type | Description                             |
| --- | ---- | --------------------------------------- |
| 1   | bool | If 'unit' is facing towards 'position'. |

---
```lua
Prediction.IsValidTarget(unit, range, from)
```

**Parameters:**

| Name             | Type       | Description                                                   |
| ---------------- | ---------- | ------------------------------------------------------------- |
| unit             | GameObject | Unit you want to test if is valid target                      |
| *optional* range | number     | The range in which you want to make sure target is in         |
| *optional* from  | position   | The position for using to check distance from unit for range. |

**Returns:**

| #   | Type | Description                 |
| --- | ---- | --------------------------- |
| 1   | bool | If target is a valid target |

## Data Types

### table `predResult`

**Attributes:**

| Name             | Type                | Description                                                                                    |
| ---------------- | ------------------- | ---------------------------------------------------------------------------------------------- |
| castPosition     | D3DXVECTOR3 OR bool | Best spot to cast spell at to hit target. False if no valid prediction.                        |
| targetPosition   | D3DXVECTOR3 OR bool | Exact predicted spot of target regardless of spell width/radius. False if no valid prediction. |
| hitChance        | number              | Probability of skillshot hitting target                                                        |
| interceptionTime | number              | Calculated time taken for spell to hit target (including spell delay)                          |

*Note: The difference between `castPosition` and `targetPosition` is that `castPosition` accounts for the unit's bounding radius & the spell's width, where as `targetPosition` does not and would represent the exact predicted position.*

---
#### Methods

```lua
predResult:heroCollision(nMaxCollision)
predResult:minionCollision(nMaxCollision)
```

**Parameters:**

| Name          | Type   | Description                                            |
| ------------- | ------ | ------------------------------------------------------ |
| nMaxCollision | number | The maximum number of collisions you care to check for |

**Returns:**

| #   | Type          | Description                                                                                         |
| --- | ------------- | --------------------------------------------------------------------------------------------------- |
| 1   | table OR bool | If no collisions, returns false. Otherwise, returns table of collisions up to `nMaxCollision` size. |

---
```lua
predResult:unitTableCollision(enemies, n)
```

**Parameters:**

| Name          | Type   | Description                                            |
| ------------- | ------ | ------------------------------------------------------ |
| enemies       | table  | Table of units you care to check collision on          |
| nMaxCollision | number | The maximum number of collisions you care to check for |

**Returns:**

| #   | Type          | Description                                                                                        |
| --- | ------------- | -------------------------------------------------------------------------------------------------- |
| 1   | table OR bool | If no collisions, returns false. Otherwise, returns table of collisions up to `nMaxCollision` size |

---
```lua
predResult:unitCollision(unit)
```

**Parameters:**

| Name | Type       | Description                                                 |
| ---- | ---------- | ----------------------------------------------------------- |
| unit | GameObject | Unit you want to check if will get in the way of skillshot. |

**Returns:**

| #   | Type | Description                                                     |
| --- | ---- | --------------------------------------------------------------- |
| 1   | bool | If skillshot would collide with this unit's predicted position. |

---
```lua
predResult:windWallCollision()
```

**Returns:**

| #   | Type | Description                                               |
| --- | ---- | --------------------------------------------------------- |
| 1   | bool | If skillshot would collide with any active Yasuo windwall |

---
### table `SpellData`

| Name                           | Type   | Description                                                                                                              |
| ------------------------------ | ------ | ------------------------------------------------------------------------------------------------------------------------ |
| type                           | string | Either **`Linear`**, **`Circular`**, or **`Cone`**                                                                       |
| delay                          | number | Number of seconds before the spell is cast **(do not include network ping, it is calculated in prediction)**             |
| speed                          | number | The speed of the spell. If nil value the speed is math.huge.                                                             |
| range                          | number | The max range between predicted pos and source. If nil value then range is math.huge                                      |
| width                          | number | [**`Linear`**] Full width of the spell as a rectangle (not to be confused with `radius` as used on other platforms)      |
| radius                         | number | [**`Circular`**] Radius of the spell as represented by a circle                                                          |
| angle                          | number | [**`Cone`**] Angle of the spell as represented by a cone                                                                 |
| *optional* ignorePing          | bool   | Set this flag to true if you do not want prediction to internally account for ping in the spell delay                    |
| *optional* forceBoundingRadius | bool   | Set this flag to true if you want to include target boundingRadius in `predResult.castPosition` regardless of spell type |

*Notes for calculating width, radius:*
- Linear
    - To calculate width, use the spell measure tool and enable bounding radius drawing. 
    - Cast your skillshot where the rectangle meets any edge of the boundingRadius circle until the max width where the spell still hits the unit.

    <img src="https://i.imgur.com/UfOtWJU.png" title="Linear spell" width="260"/>

- Circular
    - To calculate radius for circular spells, do the above but for only the center position of the unit to get the radius.
  
    <img src="https://i.imgur.com/vqFKaZE.png" title="Circular spell" width="260"/>

- Cone
  -  Do same as linear spell.

  <img src="https://i.imgur.com/bDb6Juw.png" title="Linear spell" width="260"/>

## Example Usage
*Note: This is non-functional code, needs target selector to actually use*

```lua
function OnLoad()
    if not _G.Prediction then
    	LoadPaidScript(PaidScript.DREAM_PRED)
    end
end

local MySpellData = 
{
    slot = SpellSlot.Q,
    type = "Linear",
    range = 1300,
    delay = 0.25,
    width = 120,
    speed = 2000
}
local MAX_COLLISIONS_ALLOWED = 1

AddEvent(Events.OnTick,
function()
    if myHero.isDead then return end

    if _G.Target then
        local result = _G.Prediction.GetPrediction(_G.Target, MySpellData, myHero)

        if result.castPosition and result.hitChance > .65 and not result:minionCollision(MAX_COLLISIONS_ALLOWED) then
            myHero:CastSpell(MySpellData.slot, result.castPosition)
        end
    end
end
)
```