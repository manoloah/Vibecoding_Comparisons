🔵 For Replit_GoalTracker (PWA)
Priority 1: Authentication (CRITICAL)
Implement a complete authentication system using Passport.js with local strategy. 
Requirements:
- Add user registration and login endpoints
- Hash passwords with bcrypt
- Store sessions in Redis (replace MemoryStore)
- Add authentication middleware to protect all /api/* routes
- Update all storage methods to use authenticated user's ID from session
- Remove the getFirstUser() pattern and use req.user.id instead
- Add logout endpoint
- Return proper 401/403 errors for unauthorized access
Why: Currently has NO authentication - anyone can access any data. This is a showstopper.
Priority 2: Error Boundaries & Better Error Handling
Add comprehensive error handling to the application:
- Implement React error boundaries in the client app
- Add error boundary component that catches and displays user-friendly errors
- Wrap all API routes with try-catch that return proper HTTP status codes
- Add a toast notification system (using react-hot-toast) for user feedback
- Replace all console.error with proper error logging
- Add error tracking service integration points (Sentry-ready)
- Create custom error classes for different error types
Why: Current error handling is basic - console.log only, no user feedback.
Priority 3: PWA Offline Support
Implement a service worker for offline PWA functionality:
- Add workbox for service worker generation
- Cache static assets (HTML, CSS, JS, images)
- Implement cache-first strategy for UI assets
- Add network-first strategy for API calls with offline fallback
- Show offline indicator when no connection
- Queue API requests when offline and sync when back online
- Update manifest.json with all required PWA fields
- Test "Add to Home Screen" functionality
Why: It's a PWA but has no offline support - missing the main PWA benefit.
🟠 For Bolt_GoalTracker (Mobile)
Priority 1: Component Library & Code Reusability
Refactor the app to use reusable components:
- Create a components/ folder with reusable UI components
- Extract a FormInput component (used in sign-in/sign-up)
- Create a Button component with variants (primary, secondary, outline)
- Build a Card component for consistent card layouts
- Make a LoadingSpinner component
- Create an ErrorMessage component for form validation
- Update all screens to use these new components
- Apply consistent styling through the component library
- Export components from components/index.ts for easy imports
Why: Heavy code duplication - sign-in/sign-up forms are 90% identical code.
Priority 2: Input Validation & Form Handling
Add comprehensive input validation using Zod:
- Install zod and @hookform/resolvers
- Create validation schemas in a schemas/ folder for all forms
- Add email format validation
- Enforce password requirements: min 8 chars, uppercase, number, special char
- Add length limits to text inputs (theme title max 100, description max 500)
- Validate alignment scores are 1-5
- Create a useForm custom hook that wraps validation logic
- Replace all alert() calls with proper toast notifications
- Show field-level validation errors below inputs
- Add loading states during API calls
Why: Very basic validation - just checks for empty fields, uses alerts for errors.
Priority 3: Performance Optimizations
Add performance optimizations throughout the app:
- Wrap AuthContext value in useMemo with proper dependencies
- Add useCallback to all event handlers in AuthContext
- Implement React.memo for Screen components
- Add lazy loading for tab screens using React.lazy and Suspense
- Create a useDebounce hook for text input
- Memoize expensive calculations (like filtering journal entries)
- Add code splitting by route
- Optimize image loading with expo-image
- Add performance monitoring hooks
Why: No memoization anywhere - AuthContext recreates objects on every render.
🎯 Impact Comparison
Replit Prompts Impact:
Authentication → Goes from 6/10 to 9/10 security, production-ready
Error Handling → Goes from 7/10 to 9/10 code quality, better UX
Service Worker → Goes from PWA-lite to full PWA, offline capability
Result: Production-ready web app competitive with any SaaS
Bolt Prompts Impact:
Component Library → Goes from 4/10 to 8/10 reusability, easier maintenance
Validation → Goes from 6/10 to 9/10 input validation, better security
Performance → Goes from 4/10 to 8/10 optimization, professional-grade
Result: Professional mobile app ready for scale
💡 Bonus 4th Prompts (If you have time)
Replit Bonus:
Add comprehensive testing:
- Set up Vitest with React Testing Library
- Write unit tests for all storage.ts methods
- Add integration tests for API endpoints
- Test authentication flows
- Add E2E tests with Playwright for critical user journeys
- Aim for 80%+ code coverage
- Add GitHub Actions CI to run tests on every commit
Bolt Bonus:
Add analytics and monitoring:
- Integrate Sentry for crash reporting
- Add Amplitude or Mixpanel for analytics
- Track key events: sign up, journal entry, theme creation
- Add performance monitoring
- Implement proper logging with different levels (debug, info, error)
- Create a useAnalytics hook for easy tracking
- Add user properties tracking
🏁 After These Prompts
Replit would have:
✅ Complete authentication system
✅ Production-ready security
✅ Full PWA capabilities
✅ Professional error handling
✅ Redis-backed sessions (scalable)
Bolt would have:
✅ Clean, reusable component architecture
✅ Enterprise-grade validation
✅ Optimized performance
✅ Professional code organization
✅ Better maintainability