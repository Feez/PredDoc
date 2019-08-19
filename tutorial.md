# DreamPred & DreamTS - bik tutorial

Instead of the standard API documentation which can be hard to follow, just going to code a standard skillshot champion script and explain as I go. The script is for the AP champion `Hotboi420`, coming soon™.

## Setup
```lua
if myHero.charName ~= "Hotboi420" then return end

require("FF15Menu")
require("utils")

local DreamTS = require("DreamTS")
local Orbwalker = require("FF15OL")
local Hotboi = {}

if not _G.Prediction then
    _G.LoadPaidScript(_G.PaidScript.DREAM_PRED)
end
```

We do the usual stuff of including the typical libraries, and then do `_G.LoadPaidScript` with the prediction since it is super special. 

A local table named `Hotboi` is created because I prefer to create my champion scripts in an object-oriented design for readability, and this will be the easiest way to do it.

# Additional Setup
```lua
local CastModeOptions = {"instant", "slow", "very slow"}
```
This will be used later, but I like to give the option to the user to be able to set what cast rate they want the prediction to be set at. I usually prefer `slow`, but `very slow` can be good if you want to be even more conservative with your spell casts. `instant` is trigger happy and I can't recommend for many situations.

# Initialization
```lua
function Hotboi:init()
    self.q = {
        type = "linear",
        range = 1000,
        delay = 0.25,
        width = 125,
        speed = 1400,
        collision = {
            ["Wall"] = true,
            ["Hero"] = true,
            ["Minion"] = true
        }
    }

    self:LoadMenu()
    self:LoadTS()

    AddEvent(
        Events.OnTick,
        function()
            self:OnTick()
        end
    )

    PrintChat("Hotboiii loaded.")
end
```

I split the loading process into multiple functions for the sake of this tutorial having smaller chunks of code. The functions not defined yet will be coded later.

We declare a spell on the implicit `self` table with spell data formatted how you would expect to, with the exception that the `type` of the spell should either be `linear`, `circular`, `cone`, or `targetted`. The `collision` is an optional table that should define `true` or `false` values for each of `Wall`, `Hero`, and `Minion`.

# Menu
```lua
function Hotboi:LoadMenu()
    self.menu = _G.Menu("hotboi420", "Hotboi")
    self.menu:sub("ts", "Target Selector")
    
    self.menu:sub("antigap", "Anti-Gapcloser")
    for i, enemy in ipairs(ObjectManager:GetEnemyHeroes()) do
        if not self.menu.antigap[enemy.charName] then
            self.menu.antigap:checkbox(enemy.charName, enemy.charName, true)
        end
    end

    self.menu:sub("interrupt", "Interrupter")
    _G.Prediction.LoadInterruptToMenu(self.menu.interrupt)

    self.menu:sub("spells", "Spell cast rates")
    self.menu.spells:list("q", "Q", 2, CastModeOptions)
end
```

Since this is a bik script, we're adding auto anti-gapcloser and auto interrupt. This is done pretty easily and can make your script sound like it took a ton of time to code.

- A blank sub-menu for the target selector is created, it will be used when initializing DreamTS.

- The anti-gap menu is simply created by loading evey enemy hero's name into the menu which we will check against later.

- The interrupt menu utilizes `_G.Prediction.LoadInterruptToMenu` which takes in a menu/sub-menu parameter and does the rest with loading. What it loads into the menu will be shown later.

Lastly, we add options for the spell cast rates. Since this is a single spell bik script, I just do it for Q. The value is defaulted to `2`, which is `slow` in our `CastModeOptions` list `{"instant", "slow", "very slow"}`.

# DreamTS Initialization
```lua
function Hotboi:LoadTS()
    self.TS =
        DreamTS(
        self.menu,
        {
            Damage = DreamTS.Damages.AP
        }
    )
end
```
**Short:**
Use this code with either `DreamTS.Damages.AP` or `DreamTS.Damages.AD`.

**Long:**
The `DreamTS` constructor is takes in [1] a menu/sub-menu parameter, and [2] a table. The table is to include a `Damage` field which should be a function that returns some damage based off of a unit. Internally, it takes a raw value of 200 physical or magic damage and calculates based off the units stats how much would be the final damage. Unless you care about getting super advanced with it, using either `DreamTS.Damages.AP` or `DreamTS.Damages.AD` is fine 99% of the time. 

For heros where spell damage is heavily dependent on some other condition being true, it might be worth making your own function. Such a function might be like this:
```lua
local function CalcDmg(unit)
    if SomeCondition(unit) then
        return 5 * DreamTS.Damages.AP(unit)
    end

    return 2 * DreamTS.Damages.AP(unit)
end
```

and then this function would be used as the `Damage` field.

# Helper Stuff
```lua
function Hotboi:GetCastRate(spell)
    return CastModeOptions[self.menu.spells[spell].value]
end
```
I recommend this in your code somewhere, it's just an easier way to get the menu cast rate string. Such as `self:GetCastRate("q")` -> `"very slow"`.

```lua
function Hotboi:CastQ(pred)
    if pred.rates[self:GetCastRate("q")] then
        myHero.spellbook:CastSpell(SpellSlot.Q, pred.castPosition)
        pred:draw()
        return true
    end
end
```
A cast function to return `true` if you actually cast can be beneficial, combined with drawing the prediction result.

**Note**: Always try to do `pred:draw()` when you use a prediction result that you cast on. It is never bad to do `pred:draw()`, no extra calculations are done unless debug drawing is enabled in the prediction menu.

# OnTick (Combo, Anti-Gap, Interrupt)
Before getting into the combo code, it is noteworthy to point out the signature of useful TS functions:
```lua
function TargetSelector:GetTargets(spell, source, ValidTarget, ValidPred, TargetMode)
function TargetSelector:GetTarget(spell, source, ValidTarget, ValidPred, TargetMode)
function TargetSelector:update(ValidTarget, TargetMode)
```

The difference between these functions is that `GetTarget` and `GetTargets` are reserved for using with integration with prediction. `update`, however, can be used without any prediction integration. I'll only be doing `GetTargets` to cover mostly everything at once.

```lua
function Hotboi:OnTick()
    if myHero.dead then return end

    local combo_mode  = Orbwalker:GetMode() == "Combo" and not Orbwalker:IsAttacking()

    if myHero.spellbook:CanUseSpell(SpellSlot.Q) == 0 then

        local q_targets, q_preds = self.TS:GetTargets(
                self.q, -- spell
                myHero, -- [optional] source (position or unit)
                nil,    -- [optional] ValidTarget function
                nil,    -- [optional] ValidPred function
                self.TS.Modes["Hybrid [1.0]"] -- [optional] DreamTS mode
        )
--- continue
```
**Short:**
You can probably get by with copy pasting this code for the most part.

**Long:**
Notable parameters:
- `[optional] ValidTarget function` -> filter for targets you want to consider, formatted as `ValidTarget(unit) -> boolean`.
- `[optional] ValidPred function` -> filter for prediction results you want to consider, formatted as `ValidPred(unit, pred) -> boolean`.
- `[optional] DreamTS mode` -> Any of the TS modes `{"Less Cast Priority", "Less Cast", "Closest To Hero", "Closest To Mouse", "Hybrid [1.0]"}` or a custom one. I chose `"Hybrid [1.0]"` because it is a good mix of choosing closest target and the best target from `Less Cast Priority` which is good for crowd-control spells. `Less Cast Priority` is the default for this parameter.

At this point, the variable `q_targets` is going to simply be a list of valid targets the target selector found (in order from best to worst). This **does not** mean the first target in the list is one that has a valid prediction, however. The point of that is to allow you to choose whether or not you would like to wait for a valid prediction to be found on the best target, or cast at the first best possible target right away.

`q_preds` is going to be a list mapping a unit's `networkId` -> `predResult`, where `predResult` is a prediction result returned from DreamPred. 

**Interrupt and Anti-Gap**
```lua
--continue
        for i = 1, #q_targets do
            local unit = q_targets[i]
            local pred = q_preds[unit.networkId]

            if pred then
                if (pred.targetDashing and self.menu.antigap[unit.charName]:get()) or
                (pred.isInterrupt and self.menu.interrupt[pred.interruptName]:get()) 
                then
                    if self:CastQ(pred) then
                        return
                    end
                end
            end
        end
--continue
```

This is all that needs to be done for auto anti-gap and interrupter to work. We loop through the targets, check if the pred results are valid for interrupt/dashing and then cast based on our menu settings. The relevant flags on the prediction result are `.targetDashing`, `.isInterrupt` and `.interruptName`.

**Combo**
```lua
--continue
        if ComboMode then
            local target = q_targets[1]
            if target then
                local pred = q_preds[target.networkId]
                
                if pred and self:CastQ(pred) then
                    return
                end
            end
        end
    end
end
```

