# Design Principles - WodVision

**Source**: Basato su [claude-code-workflows](https://github.com/OneRedOak/claude-code-workflows) - Ispirato a Stripe, Airbnb, Linear

---

## 🎨 Filosofia: Design Excellence

**"Great design is invisible - users shouldn't have to think"**

Design eccellente significa:
- **Intuitivo** - L'utente capisce cosa fare senza istruzioni
- **Consistente** - Pattern ripetuti in tutta l'app
- **Accessibile** - Usabile da TUTTI
- **Delightful** - Piccoli dettagli che sorprendono

---

## 🌟 Design Inspiration

### Stripe
- **Chiarezza** - Ogni azione ha un outcome chiaro
- **Fiducia** - Design che ispira sicurezza (payments)
- **Performance** - Nessun lag percepibile

### Airbnb
- **Visual hierarchy** - L'occhio va dove deve andare
- **Immagini first** - Content is king
- **Trust signals** - Review, badge, verifiche

### Linear
- **Speed** - Keyboard shortcuts, instant feedback
- **Polish** - Ogni pixel curato
- **Dark mode** - Fatto bene, non solo inversione colori

---

## 📐 Core Design Standards

### 1. Visual Hierarchy

**L'utente deve capire immediatamente cosa è importante**

```
Priorità visiva:
━━━━━━━━━━━━━━━━━━━━━━
    CTA Primaria
    (grande, colore accent)
━━━━━━━━━━━━━━━━━━━━━━
    Contenuto Principale
    (medio, peso normale)
━━━━━━━━━━━━━━━━━━━━━━
    Azioni Secondarie
    (piccolo, colore neutro)
━━━━━━━━━━━━━━━━━━━━━━
```

**Esempio WodVision:**
```dart
// ✅ GOOD - Clear hierarchy
Column(
  children: [
    // Hero: Video thumbnail (primary)
    Container(height: 200, child: VideoPlayer()),

    // Main content: Score
    Text('Form Score: 85', style: headline),

    // Secondary: Details
    Text('Analyzed 2 hours ago', style: caption),

    // CTA: Primary action
    ElevatedButton(child: Text('View Report')),

    // Secondary action
    TextButton(child: Text('Delete')),
  ],
)
```

### 2. Consistency

**Stessi pattern = Meno cognitive load**

#### Layout Consistency
- **Spacing system**: 4px, 8px, 16px, 24px, 32px, 48px
- **Border radius**: 8px per card, 16px per modals
- **Padding standard**: 16px horizontal, 24px vertical

#### Component Consistency
```dart
// ✅ GOOD - Reusable pattern
class PrimaryButton extends StatelessWidget {
  final String text;
  final VoidCallback onPressed;

  // Stesso design ovunque
  @override
  Widget build(BuildContext context) {
    return ElevatedButton(
      style: ElevatedButton.styleFrom(
        backgroundColor: AppColors.accent,
        padding: EdgeInsets.symmetric(horizontal: 32, vertical: 16),
        shape: RoundedRectangleBorder(
          borderRadius: BorderRadius.circular(8),
        ),
      ),
      onPressed: onPressed,
      child: Text(text, style: TextStyle(fontSize: 16, fontWeight: FontWeight.w600)),
    );
  }
}
```

#### Color Consistency
```dart
// Design system colors
class AppColors {
  // Primary
  static const Color accent = Color(0xFFFF6B35);  // Orange brand
  static const Color primary = Color(0xFF2B2D42);  // Dark blue

  // Semantic
  static const Color success = Color(0xFF06D6A0);
  static const Color warning = Color(0xFFFFC857);
  static const Color error = Color(0xFFEF476F);

  // Neutrals
  static const Color text = Color(0xFF1A1A1A);
  static const Color textSecondary = Color(0xFF6B6B6B);
  static const Color background = Color(0xFFFAFAFA);
}
```

### 3. Responsive Design

**Design che funziona su TUTTI i device**

```dart
// ✅ GOOD - Adaptive layout
class AdaptiveLayout extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    final width = MediaQuery.of(context).size.width;

    if (width > 600) {
      // Tablet/Desktop - Grid
      return GridView.builder(gridDelegate: ...);
    } else {
      // Mobile - List
      return ListView.builder(...);
    }
  }
}
```

**Breakpoints WodVision:**
- Mobile: < 600px (primary)
- Tablet: 600-900px (future)
- Desktop: > 900px (future web)

### 4. Accessibility (WCAG AA+)

**L'app deve essere usabile da TUTTI**

#### Color Contrast
```
Text / Background contrast ratio:
- Normal text: min 4.5:1
- Large text (18px+): min 3:1
- Icons: min 3:1
```

#### Touch Targets
```dart
// ✅ GOOD - Minimum 44x44 touch target
Container(
  width: 44,
  height: 44,
  child: IconButton(icon: Icon(Icons.delete), onPressed: ...),
)
```

#### Screen Readers
```dart
// ✅ GOOD - Semantic labels
Semantics(
  label: 'Delete video',
  button: true,
  child: IconButton(icon: Icon(Icons.delete), onPressed: ...),
)
```

#### Font Scaling
```dart
// ✅ GOOD - Respects user font size settings
Text(
  'Welcome',
  style: Theme.of(context).textTheme.headline1, // Scales automatically
)

// ❌ BAD - Fixed size
Text('Welcome', style: TextStyle(fontSize: 24)) // Ignores accessibility settings
```

---

## 🎯 Interaction Design

### Loading States

**Mai lasciare l'utente nel dubbio**

```dart
// ✅ GOOD - Clear loading states
if (isLoading) {
  return Column(
    children: [
      CircularProgressIndicator(),
      SizedBox(height: 16),
      Text('Analyzing your video...'),
      SizedBox(height: 8),
      Text('This may take 2-3 minutes', style: caption),
    ],
  );
}
```

### Empty States

**Empty ≠ Broken**

```dart
// ✅ GOOD - Helpful empty state
if (journeys.isEmpty) {
  return EmptyState(
    icon: Icons.video_library,
    title: 'No videos yet',
    description: 'Upload your first workout video to get AI-powered feedback',
    action: ElevatedButton(
      child: Text('Upload Video'),
      onPressed: () => Navigator.push(...),
    ),
  );
}
```

### Error States

**Errori che aiutano, non frustrano**

```dart
// ❌ BAD
showDialog(
  context: context,
  builder: (_) => AlertDialog(title: Text('Error'), content: Text('Something went wrong')),
);

// ✅ GOOD - Actionable error
showDialog(
  context: context,
  builder: (_) => AlertDialog(
    title: Text('Upload Failed'),
    content: Text('Your video couldn\'t be uploaded. Check your internet connection and try again.'),
    actions: [
      TextButton(child: Text('Cancel'), onPressed: () => Navigator.pop(context)),
      ElevatedButton(child: Text('Retry'), onPressed: () => retryUpload()),
    ],
  ),
);
```

### Success States

**Celebra i successi dell'utente**

```dart
// ✅ GOOD - Positive feedback
showDialog(
  context: context,
  builder: (_) => SuccessDialog(
    icon: Icons.check_circle,
    iconColor: AppColors.success,
    title: 'Analysis Complete!',
    message: 'Your deadlift form scored 85/100',
    action: ElevatedButton(
      child: Text('View Report'),
      onPressed: () => navigateToReport(),
    ),
  ),
);
```

---

## 🎨 Visual Polish

### Micro-interactions

**Piccoli dettagli che deliziano**

```dart
// ✅ GOOD - Button press feedback
GestureDetector(
  onTapDown: (_) => setState(() => _isPressed = true),
  onTapUp: (_) => setState(() => _isPressed = false),
  onTap: onPressed,
  child: AnimatedContainer(
    duration: Duration(milliseconds: 100),
    transform: Matrix4.identity()..scale(_isPressed ? 0.95 : 1.0),
    child: ...
  ),
)
```

### Animations

**Smooth > Fast**

```dart
// ✅ GOOD - Smooth page transition
Navigator.push(
  context,
  PageRouteBuilder(
    transitionDuration: Duration(milliseconds: 300),
    pageBuilder: (_, __, ___) => NextScreen(),
    transitionsBuilder: (_, animation, __, child) {
      return FadeTransition(opacity: animation, child: child);
    },
  ),
);
```

**Animation Guidelines:**
- Duration: 200-400ms (mai > 500ms)
- Easing: Ease-out per entrate, Ease-in per uscite
- Purpose: Ogni animation deve avere un motivo (guidare l'occhio, feedback)

### Shadows & Elevation

```dart
// Design system elevation
class AppElevation {
  static BoxShadow card = BoxShadow(
    color: Colors.black.withOpacity(0.08),
    blurRadius: 8,
    offset: Offset(0, 2),
  );

  static BoxShadow modal = BoxShadow(
    color: Colors.black.withOpacity(0.16),
    blurRadius: 24,
    offset: Offset(0, 8),
  );
}
```

---

## 📱 Mobile-First Patterns

### Bottom Sheets > Dialogs

```dart
// ✅ GOOD - Thumb-friendly on mobile
showModalBottomSheet(
  context: context,
  builder: (_) => SubscriptionOptions(),
);

// ❌ BAD - Hard to reach on large phones
showDialog(context: context, builder: (_) => SubscriptionDialog());
```

### Thumb Zones

```
┌─────────────────┐
│                 │  Easy reach (green)
│   ✓ Primary     │
│   ✓ Actions     │
├─────────────────┤
│                 │  Stretch reach (yellow)
│   ⚠ Secondary   │
├─────────────────┤
│   ✗ Avoid       │  Hard reach (red)
│                 │
└─────────────────┘
```

### Swipe Gestures

```dart
// ✅ GOOD - Natural mobile interaction
Dismissible(
  key: Key(journey.id.toString()),
  direction: DismissDirection.endToStart,
  background: Container(
    color: Colors.red,
    alignment: Alignment.centerRight,
    padding: EdgeInsets.only(right: 20),
    child: Icon(Icons.delete, color: Colors.white),
  ),
  onDismissed: (_) => deleteJourney(journey),
  child: JourneyCard(journey),
)
```

---

## 🧪 Design QA Checklist

### Visual Quality
- [ ] Font sizes leggibili (min 14px per body)
- [ ] Colori contrastati (WCAG AA)
- [ ] Spacing consistente (8px grid)
- [ ] Allineamenti precisi (no pixel dispari)
- [ ] Immagini non pixelate

### Interaction Quality
- [ ] Loading states per ogni async operation
- [ ] Empty states utili
- [ ] Error messages actionable
- [ ] Success feedback chiaro
- [ ] Animations smooth (60 FPS)

### Responsive Quality
- [ ] Design funziona su iPhone SE (375px)
- [ ] Design funziona su iPad (768px+)
- [ ] Landscape mode gestito
- [ ] Safe areas rispettate (notch, bottom bar)

### Accessibility
- [ ] Touch targets min 44x44
- [ ] Screen reader labels
- [ ] Color contrast verificato
- [ ] Font scaling supportato
- [ ] Keyboard navigation (web/desktop)

### Performance
- [ ] Immagini ottimizzate (< 200KB)
- [ ] Lazy loading per liste lunghe
- [ ] No jank durante scroll
- [ ] Animazioni hardware-accelerated

---

## 🎨 WodVision Brand Guidelines

### Color Palette

```dart
// Primary
Orange Brand: #FF6B35  // Energy, movimento, CrossFit
Dark Blue:    #2B2D42  // Professionalità, fiducia

// Scores
Excellent:    #06D6A0  // Green (90-100)
Good:         #118AB2  // Blue (75-89)
Fair:         #FFC857  // Yellow (60-74)
Poor:         #EF476F  // Red (< 60)

// Neutrals
Background:   #FAFAFA  // Off-white
Card:         #FFFFFF  // Pure white
Text:         #1A1A1A  // Almost black
TextSecondary:#6B6B6B  // Gray
```

### Typography

```dart
// Font: Impact/Impacted (brand)
class AppTypography {
  static TextStyle headline1 = TextStyle(
    fontFamily: 'Impacted',
    fontSize: 32,
    fontWeight: FontWeight.bold,
    color: AppColors.text,
  );

  static TextStyle body = TextStyle(
    fontFamily: 'System',  // San Francisco/Roboto
    fontSize: 16,
    fontWeight: FontWeight.normal,
    color: AppColors.text,
  );
}
```

### Iconography
- **Style**: Rounded (friendly, approachable)
- **Weight**: Regular (not too thin, not too bold)
- **Library**: Material Icons / Custom illustrations

### Voice & Tone
- **Encouraging** - "Great job! Your form is improving"
- **Direct** - "Fix your knee alignment"
- **Motivating** - "You're 85% there. Let's nail that last 15%"

---

## 📚 Design Resources

### Tools
- **Figma** - Design mockups (if needed)
- **ColorBox** - Color palette generation
- **Contrast Checker** - WCAG compliance

### Inspiration
- [Stripe Design](https://stripe.com/design)
- [Airbnb Design](https://airbnb.design/)
- [Linear Design](https://linear.app/)
- [Dribbble](https://dribbble.com/) - Fitness app patterns
- [Mobbin](https://mobbin.com/) - Mobile app screenshots

### Testing
- **Simulator** - Test on multiple screen sizes
- **Real devices** - Nothing beats physical testing
- **Lighthouse** - Web accessibility audit (future)

---

## 🎯 Design Review Process

### Before Implementation
1. **Sketch** - Quick wireframe (paper or Figma)
2. **Validate** - Does it solve the user problem?
3. **Check patterns** - Reuse existing components?
4. **Accessibility** - Can everyone use it?

### During Implementation
1. **Preview frequently** - Hot reload è tuo amico
2. **Test edge cases** - Long text, empty states, errors
3. **Test devices** - Small & large screens
4. **Polish details** - Spacing, alignment, colors

### After Implementation
1. **User test** - Watch someone use it
2. **Performance** - Check frame rate
3. **Accessibility** - Screen reader test
4. **Document** - Screenshot per future reference

---

**Remember**: "Design is not just what it looks like and feels like. Design is how it works." - Steve Jobs

*Ultimo aggiornamento: 15 Gennaio 2026*
