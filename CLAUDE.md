# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

# Project Guidelines

## Code Style

**TypeScript + Expo React Native patterns:**
- Functional components only—no class components ([themed-text.tsx](components/themed-text.tsx), [parallax-scroll-view.tsx](components/parallax-scroll-view.tsx))
- Named exports for components; type definitions using intersection types (`type ThemedTextProps = TextProps & {...}`)
- Imports use `@/*` path alias (`@/hooks/use-color-scheme`, `@/constants/theme`)
- Styling via `StyleSheet.create()` with style array composition: `style={[baseStyle, conditionalStyle, style]}`
- Platform checks: `process.env.EXPO_OS === 'ios'` or `Platform.select()` ([haptic-tab.tsx](components/haptic-tab.tsx))
- Platform-specific files: `.ios.tsx` for iOS-specific code, `.tsx` as fallback ([icon-symbol.ios.tsx](components/ui/icon-symbol.ios.tsx))

## Architecture

**Expo Router file-based routing:**
- [app/_layout.tsx](app/_layout.tsx) is root layout; [app/(tabs)/_layout.tsx](app/(tabs)/_layout.tsx) creates tab navigation
- Route groups use parentheses: `(tabs)/` excludes "tabs" from URL path
- Screens live in `app/`, reusable components in `components/`, UI primitives in `components/ui/`

**Theming system (no external state management):**
- [constants/theme.ts](constants/theme.ts) defines `Colors.light` and `Colors.dark`
- Components accept `lightColor`/`darkColor` props as overrides
- `useThemeColor()` hook resolves colors (props override theme defaults)
- `useColorScheme()` detects system preference

**Animations:**
- Use `react-native-reanimated` for 60fps animations (worklets run on UI thread)
- See [parallax-scroll-view.tsx](components/parallax-scroll-view.tsx) for `useAnimatedStyle`, `interpolate` patterns

## Build and Test

```bash
npm install              # Install dependencies
npm start                # Start Expo dev server (choose platform in terminal)
npm run android          # Launch Android
npm run ios              # Launch iOS
npm run web              # Launch web
npm run lint             # Run ESLint
npm run reset-project    # Move starter code to app-example/, create blank app/
```

**Key setup notes:**
- Expo SDK ~54.0, React 19.1.0, React Native 0.81.5
- New Architecture enabled + React Compiler (experimental)
- TypeScript strict mode in [tsconfig.json](tsconfig.json)

## Project Conventions

**Component patterns differing from defaults:**
- **Themed wrappers** ([themed-text.tsx](components/themed-text.tsx), [themed-view.tsx](components/themed-view.tsx)): Extend native components with theme props—use these instead of raw `<Text>`/`<View>` for consistency
- **Haptic feedback** ([haptic-tab.tsx](components/haptic-tab.tsx)): iOS-only via `expo-haptics`, no-op on other platforms
- **External links** ([external-link.tsx](components/external-link.tsx)): Open in-app browser on native, normal `<a>` on web—use `ExternalLink` component, not direct `Linking.openURL()`
- **Icons**: Use `IconSymbol` component ([icon-symbol.ios.tsx](components/ui/icon-symbol.ios.tsx))—renders SF Symbols on iOS, Material Icons elsewhere
- **Collapsible sections** ([collapsible.tsx](components/ui/collapsible.tsx)): Local `useState` for toggle, icon rotation via inline transforms

**Avoid:**
- Default exports for components
- Inline styles outside StyleSheet (except conditional transforms)
- Direct `useColorScheme()` for colors—use `useThemeColor()` instead
- Class components or external state libs (Redux, MobX)—this project uses local state + hooks