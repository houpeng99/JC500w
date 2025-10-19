# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-page HTML application for daily recipe/formula tracking and review system. The entire application is contained in `春哥配方复盘完整版.html` - a self-contained HTML file with embedded CSS and JavaScript.

## Application Architecture

### Data Model
The application uses a hierarchical data structure stored in `localStorage`:
```javascript
appData = {
    currentDate: string,      // Selected date
    currentScheme: string,    // 'small' | 'medium' | 'large' | 'xlarge'
    currentView: string,      // 'daily' | 'weekly'
    dailyData: {
        [date]: {
            [schemeType]: {
                ingredients: [{
                    index: number,
                    id: string,              // '001' to '045'
                    selectedHandicap: number, // -4 to +4
                    selectedOptions: number[], // [1-6]
                    hidden: boolean,
                    reviewStatus: 'unset' | 'correct' | 'incorrect',
                    correctAnswer: number | null
                }],
                result: 'pending' | 'good' | 'bad'
            }
        }
    }
}
```

### Recipe Types
- **小杯 (small)**: 2 ingredients, 2x1 format
- **中杯 (medium)**: 2 ingredients, 2x1 format
- **大杯 (large)**: 3 ingredients, 3x1 format
- **超大杯 (xlarge)**: up to 8 ingredients, mxn format (supports hiding unused ingredients)

### Key Components

**Option Selection System** (lines 807-850):
- Each ingredient has 6 selectable options (mapped to values 3/1/0)
- Options 1-3 represent upper tier (handicap 0)
- Options 4-6 represent lower tier (uses selected handicap)
- Multi-select enabled in normal mode
- In review mode with 'incorrect' status, clicking sets the correct answer

**Preview Generation** (lines 975-1012):
- Formats selected ingredients into readable text
- Separates upper tier (handicap 0) and lower tier (custom handicap) options
- Example: "配料001 数值0 选项:3,1 | 数值+2 选项:0"

**Review System** (lines 889-963):
- Mark ingredients as correct (✓) or incorrect (✗)
- Visual feedback: correct shows red highlight, incorrect shows black with correct answer in green
- Review status persists with saved data

### Two Main Views

**Daily View**:
- Select date and recipe type
- Configure ingredients (ID, handicap, options)
- Mark review status per ingredient
- Rate overall recipe as good/bad

**Weekly View**:
- Select week number (ISO week format)
- View statistics for all 4 recipe types
- See 7-day calendar grid with results
- Export CSV or generate text report

## Development Notes

### Testing the Application
Simply open `春哥配方复盘完整版.html` in any modern browser. No build process required.

### Data Persistence
All data is stored in browser `localStorage` under key `'dailySchemeData'`. To reset:
```javascript
localStorage.removeItem('dailySchemeData');
```

### Week Calculation
Uses ISO 8601 week numbering (lines 1075-1105). Monday is first day of week.

### Critical Interaction Pattern
The `toggleOption()` function has dual behavior:
- Normal mode: toggles option selection (multi-select)
- Review mode (when ingredient marked incorrect): sets correct answer (single-select)

This pattern is intentional for the review workflow.
