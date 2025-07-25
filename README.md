
# RPG ATTRIBUTES SYSTEM

---

<p align="center">
	<img src="https://www.dropbox.com/scl/fi/3uyxip1lo06d521zlkbd8/ATT.png?rlkey=2vz9rlzzmpqsjll105bz8srfy&st=wryszjnz&raw=1" alt="ATT">
</p>

---

## Introduction
This is implementation of a performant, generic, multiplayer-ready **Attribute System** for RPG / RTS projects.<br>
The system works by network replicating [***Gameplay Tags***](https://docs.unrealengine.com/4.26/en-US/ProgrammingAndScripting/Tags/) instead of reproducing the raw state of [Unreal Objects](https://docs.unrealengine.com/4.27/en-US/API/Runtime/CoreUObject/UObject/UObject/).<br>
With this setup we can create complex attribute calculations without having to rework the network code everytime game decide to change some idea of implementation details of a project.<br>
Since RPG games require a ridiculous large amount of attribute calculations, across multiple entangled systems, this is a core component for the development of any higher level system to be implemented.<br>

---

## Example Component

<div class="warning">

**NOTE:**
Sample Blueprint can be found in folder: \ATT\Content\Example\

</div>


![ATT_Sample](https://www.dropbox.com/scl/fi/x0qz5kfkaig2by55kz0qm/ATT_Sample.png?rlkey=h1qu1uhvqr1rf4zo1bk45w1md&st=zkntx6d3&raw=1)


---

## <code>UCharacterAttributeSetComponent</code> - Attributes Component

---

![ATT_Sample](https://www.dropbox.com/scl/fi/8ep0ulakniddn903emdx6/ATT_Component.png?rlkey=e82igenju0jgaknwcgw939lej&st=b7e5ynym&raw=1)

The Attribute Set Component is properly setup to create, manage the memory, and replicate the data of <code>UCharacterFeature</code> and <code>UCharacterAttribute</code> sub-objects that you create within the **Details Panel**.<br>
Although it's possible to create Blueprints of this component, it's recommended that you add the base Component Class to character blueprints instead, to avoid conflicts with Unreal's serialization system where components of instantiated blueprints can ignore changes to the base component blueprint if the fields of that instance are marked as '*Dirty*' (modified by the user). This is just a minor flaw of the Unreal Editor, but can confuse developers regularly.<br><br>

### The Attributes Component can host two sets of properties:

<code>UCharacterFeature</code> - **Character** Features are attributes that usually will serve as base attributes to more dynamic attributes. The main characteristics of these properties are that they can simply output a <code>Base Value</code> or have that same value computed by an input <code>Data Table</code>. In this case the <code>GetFormulatedValue()</code> method will automatically consider the existence of a table and return the computed value, whereas the <code>Base Value</code> access will return the base, simple, raw replicated value of the attribute...<br><br>

<code>UCharacterAttribute</code> - **Character Attributes** are a little different from **Features** in the sense that Attributes can be mapped to a <code>Curve Table</code> from its <code>Base Value</code>, instead of Tables, and can also by default be linked to adjacent Attributes with their <code>Attribute Tag</code> address. When an Attribute instance has a **Min/Max** assigned tag, its own <code>GetFormulatedValue()</code> method will automatically take that external input into consideration when generating the final value.<br><br>

---

![ATT_GetFormulatedValue](https://www.dropbox.com/scl/fi/zx806lx94tcawjehzu4io/ATT_GetFormulatedValue.png?rlkey=pa3ivghjuhysqolfixosild5e&st=t2kaobo2&raw=1)

## <code>GetFormulatedValue()</code> - Custom Fomulas

Although <code>Attribute Modifiers</code> can be blueprinted, and spawned as blueprints objects, **Attributes** and **Features** cannot!<br>
Still, there are many situations where designers are going to need or wish they just had a blueprint to develop custom attribute formulas. There's no deny the great usefulness of blueprints, but if go on converting everything to blueprints then they become a performance problem later on.<br>
To address this problem, to create custom formulas, designers can generate functions within the parent character blueprint where the function name is mapped to the Gameplay Tag address of that Attribute or Feature.<br>

> For example:

The Attribute <code>Experience</code> is mapped to a **| Level | EXP |** Data Table and, to return the correct value, the <code>GetFormulatedValue()</code> method of the EXP Attribute would have to track first the actual value of another Attribute: <code>Level</code>. The current output of the Level Attribute can be many, will depend on how many items, weapons, accessories, the character has active that can influence the final value...<br>
But the output value of **Experience** Attribute requires to know the real base value, so we can map the Attribute Tag to a custom Blueprint Function defined on the Character Blueprint itself and query the actual base value of the Level Attribute every time we have to update the display value of the **Exerience** Attribute from the Data Table automatically mapped to the base value of the **Level** Attribute:

![ATT_CustomFormula](https://www.dropbox.com/scl/fi/rboar35mpxsur0l6zdoaf/ATT_CustomFormula.png?rlkey=wuqtmj9zbbekcgow3y9rmncvu&st=b7td62v0&raw=1)

![ATT_CustomFormulaGraph](https://www.dropbox.com/scl/fi/pta46uctfs01og2t4gfkp/ATT_CustomFormulaGraph.png?rlkey=tb6fg8bz75f8xf4nco7vswmap&st=gcfb4x81&raw=1)

<br>

> * ## Pay extra attention to the <code>signature</code> of Formula Functions!
> * ## They must always follow the format of an input <code>Feature Data</code> and an output of <code>Int64</code> for <code>FEATURES</code>!
> * ## They must always follow the format of an input <code>Attribute Data</code> and an output of <code>Int64</code> for <code>ATTRIBUTES</code>!

<br>

![ATT_Signature](https://www.dropbox.com/scl/fi/8rna8v92h88i2vhw24pj1/ATT_Signature.png?rlkey=lznyy0no5wwvtfj84wwnzwq64&st=ygd1ptgf&raw=1)

![ATT_FormulaUse](https://www.dropbox.com/scl/fi/uxnnhaj2eaolmj4fuywdu/ATT_FormulaUse.png?rlkey=jkrg19m5vg68ba2lcn6q4ofzl&st=qo9vdqi7&raw=1) ![ATT_FormulaSignature](https://www.dropbox.com/scl/fi/61335dnmc6adugpitplir/ATT_FormulaSignature.png?rlkey=1io8sfhzxzbaorsv4g1v9zu8u&st=uhl55vf0&raw=1)

---

## <code>Attribute Modifiers</code> - Networked Effects

---

![ATT_MakeModifier](https://www.dropbox.com/scl/fi/872rvxo2xdl3wtysb1v8v/ATT_MakeModifier.png?rlkey=8zfolo5b4lorve2zmbraici7m&st=286sglrq&raw=1)
<br><br>
Modifiers are regular Blueprinted Objects. They cannot *Tick* and have no *World Transform*, but contain several helper functions to author any level of attribute calculation you might want to achieve.<br>

![ATT_Modifier](https://www.dropbox.com/scl/fi/8y1ru42funm0uo8rf2zbz/ATT_Modifier.png?rlkey=lb2chhxghbkckt1bvaxdhxm92&st=x63ffi1c&raw=1)

* ## **Note that you cannot apply Modifiers to an Attribute that did not register that Modifier as an Archetype!**

You must register the Archetype Class of a Modifier to the <code>ModArchetypes</code> list within the "System" section of each Attribute considering to use that Modifier sub system;<br>
Otherwise the Modifier will not work across the network:

![ATT_ModArchetypes](https://www.dropbox.com/scl/fi/q2ibnl7ms3h008by3577j/ATT_ModArchetypes.png?rlkey=5kxcc5x8uphsq617obw600qhl&st=dolt0vs8&raw=1)

![ATT_ApplyModifier](https://www.dropbox.com/scl/fi/tp43ktyprutiaffvlcanb/ATT_ApplyModifier.png?rlkey=4sz3863e4jbfde52k2farnk2k&st=986x7pc0&raw=1)

---

## <code>Attributes Entanglement</code> - Server Pre-Calculations

---

In RTS / RPG systems, very frequently, a designed rule requires an attribute to automatically update itself on the Server when another related attribute in the same component has changed.<br>
We can't escape this in modern games, but these calculations on the background can cause a networking flood if not taken care closely, one replicated attribute will trigger another and so on...<br>
To avoid networking issues, we then create several *Helper Functions* to conform each different design pattern for each type of attribute before packing a struct to send accross the network.<br>
That as well can lead to several calls to different RPCs.<br><br>
One way to avoid those problems is simply mapping *Entangled Attributes* to this pre-determined list of Base Value transformations we call *Entanglement.<br>
So instead of creating a custom *Formula* for every attribute, we tell the subsystem that every time the Base Value of the ***Level*** Atribute changes, we want the Base Value of ***Experience*** to be updated as well, at the same frame:
<br>

![ATT_Entanglement](https://www.dropbox.com/scl/fi/tkqapkga8pzvbr67ljp9z/ATT_Entanglement.png?rlkey=4d8x4vd9ztvtvdctuta4s8irk&st=079yuyaj&raw=1)
<br>
<br>
Then when we set a new value to *Level*, the Server will performs its authoritative powers to update the property value of *Experience* to everyone, so we can stop coding already at the *Set Base Value* for the Level Attribute:

![ATT_FeatureEntanglement](https://www.dropbox.com/scl/fi/f0aa739jw6nj7s91mo5ke/ATT_FeatureEntanglement.png?rlkey=tj3iyp0j8d0civs3xv6982698&st=jd1e2vdw&raw=1)

---

## <code>Attributes Preview</code> - Preview Panel

---

![ATT_Preview](https://www.dropbox.com/scl/fi/faoaqfttf30h8ry8hffri/ATT_Preview.png?rlkey=ko59c46k7y43mmevjc8zcsyvv&st=fpz78lot&raw=1)
<br><br>
To quickly check if you have got your Data Tables or Data Curves setup correctly, you can check the **Preview** from your Attribute Set Component's Details Panel and see if the values aren't exactly what you're expecting coming out from the <code>GetFormulatedValue()</code> method:
<br><br>
![ATT_Panels](https://www.dropbox.com/scl/fi/3bosghb3iyhn7j4fntv9v/ATT_Panels.png?rlkey=ya1avwse0my55gtpqs1ykqryf&st=7djpny7w&raw=1)

> ![!](https://www.dropbox.com/scl/fi/xlh9eu8vyiunaod4mkj0l/warning.png?rlkey=fqe2nvvt8jxenrkp1p44koghn&st=wuzzss9o&raw=1) Note on some versions of the Unreal Engine 5, these panels may be disabled due to an ongoing visual rework of these panels!

---

## <code>Gameplay Tags</code> - [Unreal Gameplay Tags System](https://docs.unrealengine.com/4.26/en-US/ProgrammingAndScripting/Tags/)

<br>

![ATT_Settings](https://www.dropbox.com/scl/fi/1cvi2zg1rwhsyaqjp9bxs/ATT_Settings.PNG?rlkey=btxh9x60m69r1vhnqvvr8apqc&st=p8kne70w&raw=1)

The Attribute System by default registers several default Gameplay Tags, but you can either change what is registered by editing the in the <code>ATT_AttributeData.cpp</code> source, or disable them from plugins settings:

![ATT_DefaultTags](https://www.dropbox.com/scl/fi/hiwnui1cfe6c138hogi7t/ATT_DefaultTags.PNG?rlkey=apanj6k3k6gyd7ti42b2g40hr&st=d0avgla3&raw=1)

---

# 🧰 Attribute, Feature & Modifier Sets

## 🔍 What are "Sets"?

**Sets** in this attribute system are Blueprintable `DataAsset` templates used to define **shared defaults** for:

- **Attributes**
- **Features**
- **Modifiers**
- **Traits**

They simplify prototyping and ensure consistency across multiple instances by allowing reuse of predefined configurations such as base values, gameplay tags, curves, and even UI styles.

---

## 🧱 Types of Sets

| Set Class                        | Applies To          | Data Structure Used                            |
|----------------------------------|----------------------|------------------------------------------------|
| `CharacterFeatureSet`            | Character Features   | `FHKH_CharacterFeatureData`                    |
| `CharacterAttributeSet`         | Character Attributes | `FHKH_CharacterAttributeData`                  |
| `CharacterAttributeModifierSet` | Attribute Modifiers  | `FHKH_CharacterAttributeModifierData`          |
| `CharacterTraitSet`             | Traits               | `FHKH_CharacterTraitData`                      |

Each set acts as a **container** for initialization data, styling, and specialization logic.

---

## 🧷 Attaching Sets to Attributes or Features

To attach a set to an attribute or feature:

1. Open the attribute or feature instance in the **Details Panel**
2. Locate the `Constants` slot
3. Assign an appropriate Set (e.g., `CharacterAttributeSet` for attributes)

> Once attached, the system reads from the set’s `DefaultData` and uses its parameters as initialization or fallback values.

---

## 🧪 Example Use Cases

### 🧬 Shared Feature Base

You create a `StrengthFeatureSet` with:
- `FeatureTag = ftr.strength`
- `BaseValue = 100`
- `BaseTable = CharacterStatsTable`

Attach this to all classes needing a Strength feature with the same baseline.

### ⚔️ Shared Modifier Logic

You define a `PoisonModifierSet`:
- Modifier Tag: `mod.poison`
- Lifetime: `10 seconds`
- Stackable: `True`, Stack Interval: `1 sec`

Now any attribute referencing this modifier gets the same behavior, ensuring balance across entities.

---

## 🎨 UI Support

Each set can also define:

- Custom icon (SlateBrush)
- Description text
- Color coding
- Button style for UI styling

These assets support both **editor tooling** and **runtime visualization**.

---

## ✅ Sets Summary

- **Sets** offer a clean and centralized way to manage attribute, feature, and modifier definitions
- They are ideal for **prototyping**, **consistency**, and **reusability**
- Use the `Constants` field to assign them in the editor

---

## Native Module:

To have access to the classes from native code you should import the plugin's module to your own, as usual:
```cpp
PrivateDependencyModuleNames.AddRange(new string[] { "HKH_AttributeSystem" });
```
```cpp
#include "HKH_AttributeSystem.h"
```

------

# 📘 Extended Documentation - Unreal Attribute System

## 🎮 Overview

This system implements a **performant**, **network-replicated**, and **Blueprint-integrated** attribute framework designed specifically for **RPG** and **RTS** projects in Unreal Engine. It uses **Gameplay Tags** as the primary mechanism for networking and data abstraction, reducing serialization overhead and decoupling implementation details from network logic.

## 🚀 Key Features

- 🔁 Multiplayer-Ready Replication via `GameplayTags` and subobject replication
- ⚙️ Component-Based Attribute & Feature Management
- 🧮 Customizable Attribute Formulas
- 🔗 Attribute Entanglement for Dependency Chains
- 💡 Blueprint-Friendly Modifier System
- 📊 Preview Tools for Debugging & Balancing

## 🧩 Core Classes & Concepts

### 🧱 CharacterAttributeSetComponent

Main actor component that manages both `Features` and `Attributes`. Should be attached directly to Character Blueprints instead of subclassing in Blueprints to avoid Unreal's serialization inconsistencies.

#### Replicated Properties
- `TArray<CharacterFeature*> Features`
- `TArray<CharacterAttribute*> Attributes`

#### Query Functions
```cpp
CharacterFeature* GetFeatureByTag(FGameplayTag Tag) const;
CharacterAttribute* GetAttributeByName(FName Name) const;
```

### 🧬 CharacterAttribute

Defines an attribute (e.g., Health, Mana, XP). Fully network-replicated with support for custom logic, entanglement, traits, and modifiers.

#### Core Properties
```cpp
FName AttributeName;
FGameplayTag AttributeTag;
int64 BaseValue;
FCurveTableRowHandle BaseCurve;
TMap<FGameplayTag, EntanglementType> Entanglement;
```

#### Formula Support
Custom Blueprint function:
```cpp
int64 att.health(const CharacterAttributeData& Data)
{
    return Data.BaseValue + 100; // Example
}
```

### 🔁 Attribute Entanglement

Allows automatic updates of dependent attributes when a source attribute changes.
```cpp
Entanglement.Add(Tag_Level, OnBaseValueChange);
```

### 🧱 CharacterFeature

Used for simpler or foundational attributes like Strength. Can drive complex attributes.

## 🧩 Modifiers System

Modifiers are Blueprint-spawned, non-Ticking UObject instances with formula logic to alter attributes dynamically (buffs, debuffs).

#### Lifecycle
```cpp
bool AddModifier(FGameplayTag Tag);
bool RemoveModifier(FGameplayTag Tag);
```

#### Registration
```cpp
TMap<FGameplayTag, TSubclassOf<CharacterAttributeModifier>> ModArchetypes;
```

## 🧠 Traits System

Traits represent status effects or toggles, implemented via `GameplayTags`.

#### Interface
```cpp
bool AddTrait(FGameplayTag Tag);
bool IsTraitActive(FGameplayTag Tag) const;
```

## 🧪 Preview System

Use the editor details panel to preview formulas, modifiers, and values dynamically.

## ⚙️ Integration Guide

### Module Setup
```cpp
PrivateDependencyModuleNames.AddRange(new string[] { "ATT" });
#include "ATT.h"
```

## 🔖 Gameplay Tags System

Modify tags in `AttributeData.cpp` or disable default tags via Plugin Settings.

## 📊 Class Diagram

![ATT_Classes](https://www.dropbox.com/scl/fi/1usht8begrjilkgqe2gxo/ATT_Classes.png?rlkey=9aumpnbe694u0gzwhyue5ipm5&st=wiv52dq0&raw=1)

---

## ✅ Best Practices

- Avoid Blueprint subclassing of `AttributeSetComponent`
- Use consistent `GameplayTags`
- Register modifiers with `ModArchetypes`
- Prevent circular entanglements
- Prefer Blueprint formula functions for design flexibility

---

# 📈 (BONUS) EXP / Level ➜ Attack Power Relationship

## 🧱 System Setup Overview

In this system:

- `Experience` is typically a `Character Feature` that tracks progress.
- `Level` is another `Character Feature` that is **derived** from `Experience`.
- `Attack Power` is often a `Character Attribute`, but it depends on `Level` to scale.

This means the flow of influence looks like:

```mermaid
flowchart LR
    A[Experience] --> B(Level)
    B --> C(Attack Power)
```

---

## 🔗 Step 1: Entangling EXP with Level

To ensure **Level updates when Experience changes**, use **entanglement**:

```cpp
// In Experience attribute
Entanglement.Add(FGameplayTag("Attribute.Status.LV"), EHKH_Entanglement::OnBaseValueChange);
```

This tells the system:
> “Every time `Experience` is updated, recalculate `Level`'s value.”

---

## 🧮 Step 2: Deriving Level from EXP (Custom Formula)

Create a custom **Blueprint** function in the Character Blueprint named:

```plaintext
Attribute::Status::LV
```

**Function Signature** (required by system):

```cpp
int64 Attribute::Status::LV(const FCharacterAttributeData& Data)
```

**Example Logic**:

Use a `DataTable` mapping EXP thresholds to levels.

```plaintext
If EXP = 5500 ➜ LEVEL = 7
If EXP = 7800 ➜ LEVEL = 9
```

This Blueprint custom formula reads the EXP value and looks up the level using a table like: `XPToLevelTable`.

---

## ⚔️ Step 3: Scaling Attack Power by Level

Now create another Blueprint function named:

```plaintext
Attribute::Status::ATK
```

This is your **custom formula** for Attack Power.

**Function Signature**:

```cpp
int64 Attribute::Status::ATK(const FCharacterAttributeData& Data)
```

**Logic Example**:

```cpp
int64 Level = GetAttributeByTag("Attribute.Status.LV")->GetFormulatedValue();
return Level * 10 + 25;
```

This means:
> “Attack Power increases linearly with Level. Each level gives +10 Attack Power, base value is 25.”

---

## 🔄 Real-Time Data Flow:

1. `Experience` is updated (e.g., via `AddModifier` or `SetBaseValue`)
2. `Experience` triggers recalculation of `Level` via entanglement
3. `Level` recalculates `Attack Power` through its own formula
4. UI updates based on `GetFormulatedValue()` of `Attack Power`

---

## ✅ Summary

| Attribute       | Role             | Formula Source                         | Depends On          |
|-----------------|------------------|----------------------------------------|---------------------|
| Experience      | Progress Tracker | Raw value                              | —                   |
| Level           | Milestone Marker | Blueprint formula using EXP            | Experience          |
| Attack Power    | Combat Stat      | Blueprint formula using Level          | Level               |

---