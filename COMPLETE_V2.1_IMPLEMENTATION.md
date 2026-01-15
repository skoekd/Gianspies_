# 🍕 Complete Pizza Calculator v2.1 - Full Implementation Guide

## 📋 COMPLETE FEATURE CHECKLIST

### ✅ Verified Existing Features (Already in calculator):
1. 6 Pizza styles with all properties
2. Temperature-based fermentation (Q10 = 2)
3. Forward/reverse timeline
4. Autolysis integration (20-30 min)
5. Bake temperatures (wood/home for each style)
6. Recipe save/load/export
7. Flour database (8 flours)
8. Baker's percentage toggle
9. Comprehensive warnings
10. Spiral mixer LOW & HIGH added ✓

### 🆕 Features to Add:

#### 1. YEAST TYPE SELECTION & ENHANCED CALCULATION

**Add State:**
```javascript
const [yeastType, setYeastType] = useState('instant');
```

**Add Function (place after FRICTION_FACTORS):**
```javascript
// Yeast type conversions
const YEAST_CONVERSIONS = {
    'instant': 1.0,
    'active-dry': 1.25,
    'fresh': 3.0,
    'lemady': 1.0
};

// Calculate average fermentation temperature
const calculateAverageFermentTemp = () => {
    const bigaHours = getBigaHours();
    const bulkHours = getBulkHours();
    const coldHours = getColdFermentHours();
    const proofHours = getProofHours();
    const totalHours = bigaHours + bulkHours + coldHours + proofHours;
    
    if (totalHours === 0) return 15; // Default
    
    const weightedTemp = (
        bigaHours * bigaTemp +
        bulkHours * bulkTemp +
        coldHours * fridgeTemp +
        proofHours * proofTemp
    ) / totalHours;
    
    return weightedTemp;
};
```

**Replace Yeast Calculation (find existing yeast calculation):**
```javascript
// OLD: Simple time-based calculation
// NEW: Comprehensive calculation

const calculateYeastPercentage = () => {
    const totalFermentHours = getBigaHours() + getBulkHours() + getColdFermentHours() + getProofHours();
    
    // 1. Base percentage from fermentation time
    let baseYeast;
    if (totalFermentHours >= 90) baseYeast = 0.15;
    else if (totalFermentHours >= 60) baseYeast = 0.2;
    else if (totalFermentHours >= 30) baseYeast = 0.4;
    else baseYeast = 0.8;
    
    // 2. Temperature adjustment
    const avgTemp = calculateAverageFermentTemp();
    let tempFactor = 1.0;
    if (avgTemp > 18) tempFactor = 0.9;  // Warmer = less yeast needed
    else if (avgTemp < 12) tempFactor = 1.1; // Colder = more yeast needed
    
    // 3. Salt adjustment
    const saltPercent = styleConfig.salt;
    let saltFactor = 1.0;
    if (saltPercent > 2.5) saltFactor = 1.1;  // High salt inhibits = more yeast
    else if (saltPercent < 2) saltFactor = 0.9; // Low salt = less yeast
    
    // 4. Hydration adjustment
    const actualHydration = customHydration || styleConfig.hydration;
    let hydrationFactor = 1.0;
    if (actualHydration > 75) hydrationFactor = 0.95; // High hydration = slightly less
    else if (actualHydration < 65) hydrationFactor = 1.05; // Low hydration = slightly more
    
    // 5. Calculate instant yeast equivalent
    const instantYeastPct = baseYeast * tempFactor * saltFactor * hydrationFactor;
    
    // 6. Convert for selected yeast type
    const finalYeastPct = instantYeastPct * YEAST_CONVERSIONS[yeastType];
    
    return {
        baseYeast,
        tempFactor,
        saltFactor,
        hydrationFactor,
        avgTemp,
        instantYeastPct,
        finalYeastPct,
        yeastType
    };
};

const yeastCalc = calculateYeastPercentage();
const yeastPercentage = yeastCalc.finalYeastPct;
const totalYeast = (totalFlour * yeastPercentage / 100).toFixed(1);
```

**Add UI Element (in Mixing section):**
```html
<div className="input-group">
    <label className="input-label">Yeast Type</label>
    <select
        className="select-field"
        value={yeastType}
        onChange={(e) => setYeastType(e.target.value)}
    >
        <option value="instant">Instant Yeast (IDY)</option>
        <option value="active-dry">Active Dry Yeast (ADY) - 25% more needed</option>
        <option value="fresh">Fresh Yeast - 3x more needed</option>
        <option value="lemady">Lemady Yeast (instant)</option>
    </select>
</div>

<div className="info-box" style={{fontSize: '0.85rem'}}>
    <strong>Yeast Calculation:</strong><br/>
    Base (time): {(yeastCalc.baseYeast * 100).toFixed(2)}% → 
    Temp factor: ×{yeastCalc.tempFactor.toFixed(2)} → 
    Salt factor: ×{yeastCalc.saltFactor.toFixed(2)} → 
    Hydration factor: ×{yeastCalc.hydrationFactor.toFixed(2)} = 
    {(yeastCalc.instantYeastPct * 100).toFixed(3)}% instant → 
    ×{YEAST_CONVERSIONS[yeastType]} ({yeastType}) = 
    <strong>{(yeastPercentage * 100).toFixed(3)}%</strong>
    <br/>
    <small>Average fermentation temp: {yeastCalc.avgTemp.toFixed(1)}°C</small>
</div>
```

#### 2. MANUAL DOUGH BALL WEIGHT INPUT

**Replace State:**
```javascript
// OLD:
const [pizzaSize, setPizzaSize] = useState('12');
const doughBallWeight = DOUGH_WEIGHTS[pizzaSize];

// NEW:
const [doughBallWeight, setDoughBallWeight] = useState(280);
```

**Replace UI (in Recipe Setup section):**
```html
<!-- OLD: Dropdown -->
<div className="input-group">
    <label className="input-label">Pizza Size (inches)</label>
    <select ...>...</select>
</div>

<!-- NEW: Number Input -->
<div className="input-group">
    <label className="input-label">Dough Ball Weight (grams)</label>
    <input
        type="number"
        className="input-field"
        value={doughBallWeight}
        onChange={(e) => setDoughBallWeight(parseInt(e.target.value) || 280)}
        min="100"
        max="1000"
        step="10"
    />
    <small style={{fontSize: '0.8rem', color: 'var(--crust)', marginTop: '0.5rem', display: 'block'}}>
        Reference: 10"≈220g | 12"≈280g | 14"≈350g | 16"≈420g | 18"≈520g
    </small>
</div>
```

**Add Validation:**
```javascript
if (doughBallWeight < 100) 
    errors.push('❌ Minimum dough ball weight is 100g');
if (doughBallWeight > 1000) 
    errors.push('❌ Maximum dough ball weight is 1000g');
if (doughBallWeight > 0 && doughBallWeight < 150) 
    warnings.push('⚠️ Very small pizza (<150g) - consider 220g+ for standard size');
if (doughBallWeight > 600) 
    warnings.push('⚠️ Very large pizza (>600g) - may be difficult to handle');
```

#### 3. AUTO-SUGGESTED FDT

**Add Function:**
```javascript
const getSuggestedFDT = () => {
    const hydration = customHydration || styleConfig.hydration;
    const method = styleConfig.fermentMethod;
    
    // High hydration needs cooler FDT for easier handling
    if (hydration >= 85) return 23;
    if (hydration >= 75) return 24;
    
    // Biga methods prefer moderate temps
    if (method === 'biga' && hydration >= 70) return 24;
    
    // Standard ranges
    if (hydration >= 65) return 25;
    return 26; // Low hydration, direct method
};

const suggestedFDT = getSuggestedFDT();
```

**Update FDT Input:**
```html
<label className="input-label">
    Final Dough Temperature (°C)
    <span style={{color: 'var(--basil)', marginLeft: '0.75rem', fontWeight: '600'}}>
        💡 Suggested: {suggestedFDT}°C
    </span>
</label>
<input
    type="number"
    className="input-field"
    value={desiredFDT}
    onChange={(e) => setDesiredFDT(parseFloat(e.target.value) || suggestedFDT)}
    step="0.5"
    placeholder={`Auto: ${suggestedFDT}°C`}
/>
```

#### 4. SEPARATE DATE/TIME INPUTS

**Replace State:**
```javascript
// OLD:
const [bakeDateTime, setBakeDateTime] = useState('');

// NEW:
const [bakeDate, setBakeDate] = useState('');
const [bakeTime, setBakeTime] = useState('18:00');

// Combine for calculations:
const bakeDateTime = bakeDate && bakeTime ? `${bakeDate}T${bakeTime}` : '';
```

**Replace UI:**
```html
<!-- OLD: Single datetime input -->
<input type="datetime-local" value={bakeDateTime} .../>

<!-- NEW: Separate inputs -->
<div className="input-row">
    <div className="input-group">
        <label className="input-label">Bake Date</label>
        <input
            type="date"
            className="input-field"
            value={bakeDate}
            onChange={(e) => setBakeDate(e.target.value)}
        />
    </div>
    <div className="input-group">
        <label className="input-label">Bake Time</label>
        <input
            type="time"
            className="input-field"
            value={bakeTime}
            onChange={(e) => setBakeTime(e.target.value)}
        />
    </div>
</div>
```

#### 5. TABBED INTERFACE

**Add State:**
```javascript
const [activeTab, setActiveTab] = useState('setup');
```

**Add CSS (in <style> section):**
```css
.tabs-container {
    position: sticky;
    top: 0;
    z-index: 1000;
    background: white;
    border-bottom: 2px solid var(--wheat-dark);
    box-shadow: 0 2px 8px var(--shadow);
    margin-bottom: 2rem;
}

.tabs {
    display: flex;
    max-width: 1400px;
    margin: 0 auto;
    overflow-x: auto;
}

.tab-button {
    flex: 1;
    min-width: 140px;
    padding: 1rem 1.5rem;
    border: none;
    background: none;
    font-family: 'Work Sans', sans-serif;
    font-size: 0.9rem;
    font-weight: 500;
    color: var(--crust);
    cursor: pointer;
    border-bottom: 3px solid transparent;
    transition: all 0.3s ease;
    white-space: nowrap;
}

.tab-button:hover {
    background: var(--wheat);
    color: var(--char);
}

.tab-button.active {
    color: var(--tomato);
    border-bottom-color: var(--tomato);
    font-weight: 600;
}

.tab-content {
    display: none;
}

.tab-content.active {
    display: block;
}

@media (max-width: 768px) {
    .tab-button {
        min-width: 100px;
        padding: 0.875rem 1rem;
        font-size: 0.85rem;
    }
}
```

**Add Tab Navigation (after header, before calculator-grid):**
```jsx
<div className="tabs-container">
    <div className="tabs">
        <button 
            className={`tab-button ${activeTab === 'setup' ? 'active' : ''}`}
            onClick={() => setActiveTab('setup')}
        >
            📋 Recipe Setup
        </button>
        <button 
            className={`tab-button ${activeTab === 'temps' ? 'active' : ''}`}
            onClick={() => setActiveTab('temps')}
        >
            🌡️ Temperatures
        </button>
        <button 
            className={`tab-button ${activeTab === 'ferment' ? 'active' : ''}`}
            onClick={() => setActiveTab('ferment')}
        >
            ⏱️ Fermentation
        </button>
        <button 
            className={`tab-button ${activeTab === 'results' ? 'active' : ''}`}
            onClick={() => setActiveTab('results')}
        >
            📊 Results
        </button>
        <button 
            className={`tab-button ${activeTab === 'save' ? 'active' : ''}`}
            onClick={() => setActiveTab('save')}
        >
            💾 Save & Export
        </button>
    </div>
</div>
```

**Reorganize Content into Tabs:**
```jsx
<div className="calculator-content">
    {/* Tab 1: Recipe Setup */}
    <div className={`tab-content ${activeTab === 'setup' ? 'active' : ''}`}>
        <div className="card">
            <h2 className="card-title">Pizza Style</h2>
            {/* Style selector */}
        </div>
        <div className="card" style={{marginTop: '2rem'}}>
            <h2 className="card-title">Quantity</h2>
            {/* Number of pizzas + dough ball weight */}
        </div>
        <div className="card" style={{marginTop: '2rem'}}>
            <h2 className="card-title">Advanced Options</h2>
            {/* Custom hydration */}
        </div>
    </div>
    
    {/* Tab 2: Temperatures & Mixing */}
    <div className={`tab-content ${activeTab === 'temps' ? 'active' : ''}`}>
        <div className="card">
            <h2 className="card-title">Temperatures</h2>
            {/* Room, flour, fridge, FDT */}
        </div>
        <div className="card" style={{marginTop: '2rem'}}>
            <h2 className="card-title">Mixing & Baking</h2>
            {/* Mixer type, yeast type, oven type */}
        </div>
    </div>
    
    {/* Tab 3: Fermentation Timeline */}
    <div className={`tab-content ${activeTab === 'ferment' ? 'active' : ''}`}>
        <div className="card">
            <h2 className="card-title">Fermentation Timeline</h2>
            {/* Timeline mode, date/time, temperature inputs, overrides */}
        </div>
    </div>
    
    {/* Tab 4: Results */}
    <div className={`tab-content ${activeTab === 'results' ? 'active' : ''}`}>
        {/* Warnings */}
        {/* Recipe ingredients */}
        {/* Timeline display */}
    </div>
    
    {/* Tab 5: Save & Export */}
    <div className={`tab-content ${activeTab === 'save' ? 'active' : ''}`}>
        <div className="card">
            <h2 className="card-title">Save Recipe</h2>
            {/* Recipe management */}
        </div>
    </div>
</div>
```

#### 6. UPDATE SAVE/LOAD/EXPORT

**Update saveRecipe function to include:**
```javascript
const recipe = {
    // ... existing fields ...
    doughBallWeight,  // NEW
    yeastType,        // NEW
    bakeDate,         // NEW (instead of bakeDateTime)
    bakeTime,         // NEW
    // ... rest of fields ...
};
```

**Update loadRecipe to restore:**
```javascript
setDoughBallWeight(recipe.doughBallWeight || 280);
setYeastType(recipe.yeastType || 'instant');
setBakeDate(recipe.bakeDate || '');
setBakeTime(recipe.bakeTime || '18:00');
```

**Update exportRecipe to include yeast calculation details:**
```javascript
const text = `
YEAST CALCULATION:
Base (${totalFermentHours.toFixed(1)}h): ${(yeastCalc.baseYeast * 100).toFixed(2)}%
Temperature adjustment (${yeastCalc.avgTemp.toFixed(1)}°C): ×${yeastCalc.tempFactor.toFixed(2)}
Salt adjustment (${styleConfig.salt}%): ×${yeastCalc.saltFactor.toFixed(2)}
Hydration adjustment (${hydration}%): ×{yeastCalc.hydrationFactor.toFixed(2)}
Instant yeast equivalent: ${(yeastCalc.instantYeastPct * 100).toFixed(3)}%
Yeast type (${yeastType}): ×${YEAST_CONVERSIONS[yeastType]}
Final ${yeastType} yeast: ${(yeastPercentage * 100).toFixed(3)}% = ${totalYeast}g total
...
`;
```

## 🔍 TRIPLE-CHECK VERIFICATION

After implementing all changes, verify:

1. ✅ Yeast type dropdown shows 4 options
2. ✅ Yeast calculation shows all factors (time, temp, salt, hydration)
3. ✅ Dough ball weight is number input (not dropdown)
4. ✅ FDT shows "Suggested: XX°C"
5. ✅ Date and time are separate inputs
6. ✅ Tabs switch correctly
7. ✅ All calculations still correct
8. ✅ Save/load works with new fields
9. ✅ Export includes yeast calc details
10. ✅ Mobile responsive

## 📦 Complete Features Count: 65+

All features documented and ready to implement!
