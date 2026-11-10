# Wangku Project Documentation

## Overview
Wangku is a student-focused financial education app built with Flet (Flutter for Python). It helps students track allowances, expenses, savings goals, and provides AI-powered financial coaching. The app uses a local-first approach with optional backend integration for enhanced AI capabilities.

## Project Structure

```
UpdatedWangku/
├── src/                    # Main application source code
│   ├── main.py            # Application entry point and routing
│   ├── finance.py         # Financial calculation utilities
│   ├── storage.py         # Local data persistence
│   ├── chat_client.py     # Backend communication transport
│   ├── ai_coach.py        # Local AI financial coach implementation
│   ├── shop_data.py       # Product catalog for student shop
│   ├── theme.py           # UI theming and styling
│   ├── views/             # UI screens/views
│   │   ├── dashboard.py   # Main app tabs (Home, Transactions, Savings, Coach, Shop, Profile)
│   │   ├── auth.py        # Authentication screens (welcome, registration)
│   │   ├── onboarding.py  # Initial setup flow
│   │   ├── scanner.py     # Receipt scanning camera interface
│   │   └── shop.py        # Student shop interface
│   └── components/        # Reusable UI components
│       └── brand.py       # Branding elements (logo, mascot, badges)
└── backend/               # Optional Python backend for AI coaching
    ├── server.py          # HTTP server for coach requests
    ├── gemini_coach.py    # Gemini AI integration
    ├── gemini_service.py  # Compatibility shim
    └── finance.py         # Shared financial utilities
```

## Core Files

### `src/main.py` - Application Entry Point & State Management
**Primary Functions:**
- Initializes Flet app with "Wangku" title and theme
- Manages global application state (user data, transactions, goals, UI state)
- Handles routing between screens: welcome, register, onboarding, app, scanner, receipt_review, admin
- Implements navigation bar with 6 tabs: Home, Transactions, Savings, Coach, Shop, Profile
- Manages camera integration for receipt scanning
- Handles authentication (login, registration, demo mode)
- Persists data using LocalStore
- Coordinates between all views and business logic

**Key State Variables:**
- User profile (name, role, allowance, frequency)
- Financial data (spent, balance, transactions, goals)
- Chat state for AI coach
- Camera state for receipt scanning
- Onboarding progress

### `src/finance.py` - Financial Calculation Utilities
**Functions:**
- `money(value)` - Format numbers as Malaysian Ringgit (RM X,XXX.XX)
- `number(value)` - Safely parse string to float, handling commas
- `goal_progress(saved, target)` - Calculate progress percentage (0.0-1.0)
- `monthly_allowance(amount, frequency)` - Convert weekly/monthly to monthly equivalent
- `months_to_goal(price, saved, monthly_saving)` - Estimate months to reach savings target
- `category_totals(transactions)` - Sum expenses by category
- `coach_allocation(available)` - 50/30/20 budget split (savings/flexible/buffer)
- `distribute_savings(goals, amount)` - Evenly distribute savings across incomplete goals

### `src/storage.py` - Local Data Persistence
**Class: `LocalStore`**
- Wrapper around Flet's SharedPreferences
- Methods: `get(key, default)`, `set(key, value)`, `remove(key)`
- Keys prefixed with "moneymind." for namespace isolation
- Async interface for non-blocking operations

### `src/chat_client.py` - Backend Communication Transport
**Functions:**
- `validate_https_base(url)` - Validate backend URL security
- `post_backend(contents, url, token, context, timeout)` - Send chat to authenticated backend
- `post_gemini(contents, system_instruction, model, timeout)` - Direct Gemini API calls
- `ChatTransportError` - Standardized error types for transport failures

### `src/ai_coach.py` - Local AI Financial Coach
**Primary Functions:**
- `local_reply(state, messages)` - Rule-based Malay responses using financial data
- `reply(state, messages)` - Main entry point; tries backend first, falls back to local
- `build_payload(state, messages)` - Prepare context for AI (allowance, spending, goals, categories)
- Intent recognition for: balance, allowance, purchases, transactions, categories, budgeting, savings goals, tips

**Capabilities:**
- Answers in Bahasa Melayu only
- Context-aware using local financial data
- Handles: account info, balance checks, spending analysis, affordability checks, savings planning, 50/30/20 budgeting
- Privacy-first: no personal data sent without backend configuration

### `src/shop_data.py` - Product Catalog
**Data:**
- `PRODUCTS` list with student-friendly items (backpack, notebooks, calculator, stationery)
- Each product: name, description, price, image, tag

### `src/theme.py` - UI Theming System
**Constants:**
- Color palette: BG, SURFACE, INK, MUTED, PRIMARY, PURPLE_SOFT, etc.
- Semantic colors: CORAL (expenses), MINT (balance), BLUE (transport), YELLOW (warnings)

**Helper Functions:**
- `label(text, size, color, weight)` - Styled text
- `title(text, size, color)` - Heading text
- `card(content, color, padding, border_color)` - Consistent card containers
- `primary_button(text, on_click, icon)` - Primary action buttons

## View Files (`src/views/`)

### `dashboard.py` - Main Application Tabs
**Functions:**
- `home()` - Dashboard with overview cards, spending/savings chart, savings goal, recent transactions, coach promo, shop link
- `transactions()` - Transaction list with scanner button, delete/clear options, add expense
- `savings()` - Goals management with progress bars, add/edit/delete/select, contribute savings
- `coach()` - Chat interface with suggested questions, message history, input field
- `profile()` - User profile, settings links, logout, reset onboarding
- `admin_home()` - Admin preview (placeholder)
- `transaction_row()` - Reusable transaction display component
- `overview_card()` - Metric display cards
- `money_chart()` - Switchable pie chart (spending categories vs 50/30/20 plan)

### `auth.py` - Authentication Screens
**Functions:**
- `welcome()` - Login screen with username/password, register link, demo mode
- `registration()` - Student account creation with validation

### `onboarding.py` - Initial Setup Flow (3 Steps)
**Function: `onboarding(step, values, callbacks)`**
- Step 0: Allowance amount + frequency (weekly/monthly)
- Step 1: Savings target + current saved amount
- Step 2: Dream item + price + timeline (months)
- Progress indicator, validation, contextual tips

### `scanner.py` - Receipt Scanning Interface
**Functions:**
- `live(camera, status, on_capture, on_cancel)` - Camera preview with capture button
- `review(photo, description, amount, category, message, on_save, on_retake, on_cancel)` - Post-capture review with form fields

### `shop.py` - Student Shop
**Function: `screen(on_back)`**
- Displays product catalog from shop_data.py
- Shows product images, descriptions, prices
- "Coming soon" purchase buttons

## Component Files (`src/components/`)

### `brand.py` - Branding Elements
**Functions:**
- `wordmark(size, color)` - "Wangku" logo with icon
- `mascot(size)` - Character illustration
- `badge(text)` - "Jurulatih Wangku" labeled badge

## Backend Files (`backend/`)

### `server.py` - HTTP Coach Server
**Features:**
- ThreadingHTTPServer on configurable port (default 8080)
- HMAC token authentication (WANGKU_ACCESS_TOKENS env var)
- Rate limiting: 60 requests/hour, 6 requests/minute per token
- Concurrent request gate (max 3 simultaneous)
- Request validation and sanitization
- Health endpoint at `/health`
- Coach endpoint at `/coach` (POST)
- Integrates with gemini_coach.py for AI responses

### `gemini_coach.py` - Gemini AI Integration
**Functions:**
- `build_payload(state, messages)` - Format request with financial context
- `_request(payload)` - Call Gemini API with error handling
- `reply(state, messages)` - Async wrapper for non-blocking calls
- Uses GEMINI_API_KEY and GEMINI_MODEL environment variables
- Malaysian financial coach system prompt (Bahasa Melayu)

### `gemini_service.py` - Compatibility Shim
- Re-exports from gemini_coach for server.py imports

### `finance.py` - Shared Financial Utilities
- Duplicate of src/finance.py for backend independence

## Data Flow

1. **App Launch** → `main.py` initializes → shows splash → routes to welcome/onboarding/app
2. **Authentication** → `auth.py` handles login/register → stores credentials in LocalStore
3. **Onboarding** → `onboarding.py` collects financial setup → persists to storage
4. **Main App** → `dashboard.py` tabs show financial overview
5. **Transactions** → Add manually or via `scanner.py` camera → stored in state + persisted
6. **Savings** → Goals managed in `dashboard.py` savings tab → `finance.py` distributes contributions
7. **AI Coach** → `dashboard.py` coach tab → `ai_coach.py` processes → local or `chat_client.py` → backend
8. **Shop** → `shop.py` displays `shop_data.py` products

## Key Design Patterns

- **State Centralization**: All state in `main.py` state dict, passed to views
- **Callback-Driven**: Views receive callback functions for actions
- **Local-First**: Works offline; backend optional for enhanced AI
- **Malay-First UI**: All user-facing text in Bahasa Melayu
- **Student-Focused**: Terminology, amounts, goals tailored for students
- **Privacy-Conscious**: No external APIs without explicit configuration

## Environment Variables

**Backend (.env or server environment):**
- `WANGKU_ACCESS_TOKENS` - Comma-separated auth tokens
- `GEMINI_API_KEY` - Google Gemini API key
- `GEMINI_MODEL` - Model name (default: gemini-flash-latest)
- `PORT` - Server port (default: 8080)

**Frontend:** No required environment variables (uses local storage)