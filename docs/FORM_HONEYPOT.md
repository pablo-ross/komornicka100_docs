# Form Protection Implementation

This document explains the anti-bot protection mechanisms implemented for forms in the Komornicka 100 application.

## Overview

We've implemented a multi-layered approach to prevent automated submissions to our registration and unregistration forms. This solution is lightweight, doesn't require external services, and maintains a good user experience.

## Components

### 1. Honeypot Field

The `Honeypot` component (`components/ui/Honeypot.tsx`) adds:

- An invisible field named "website" that humans won't see but bots will typically fill out
- A timestamp field to track when the form was loaded, used to detect submissions that are too quick

```tsx
<Honeypot />
```

### 2. Math Challenge

The `MathChallenge` component (`components/ui/MathChallenge.tsx`) presents a simple addition problem:

- Generates two random numbers (0-9)
- Requires the user to enter the correct sum
- Validates the answer in real-time
- Integrates with the form's submission validation

```tsx
<MathChallenge setIsValid={setIsMathChallengeValid} />
```

### 3. Form Validation

The form validation utility (`utils/formValidation.ts`) provides a function that:

- Checks if the honeypot field is empty (should be for real users)
- Verifies the form wasn't submitted too quickly (< 3 seconds) or after too long (> 30 minutes)
- Validates the math challenge response if enabled

```tsx
const validation = validateFormSubmission(formData, { validateMathChallenge: true });
```

## Integration with Forms

The protection measures have been integrated into both forms:

1. **Register Form** (`pages/register.tsx`):
   - Added the Honeypot component
   - Added the MathChallenge component
   - Added validation logic to the onSubmit function
   - Added state to track if the math challenge is valid

2. **Unregister Form** (`pages/unregister.tsx`):
   - Same integration as the registration form

## How It Works

1. When a user loads the form, the timestamp is recorded and a random math problem is generated
2. The user fills out the form and solves the math problem
3. On submission, the form data is validated:
   - If the honeypot field is filled, the submission is rejected
   - If the form was submitted too quickly or after too long, it's rejected
   - If the math challenge wasn't completed correctly, it's rejected
4. Only submissions that pass all checks are sent to the API

## Translations

New translations have been added for the security challenge UI elements in `translations/pl.ts`.

## Customizing Protection Level

If you need to adjust the protection level:

1. **Enable/Disable Math Challenge**: You can remove the MathChallenge component and its validation if you prefer a more streamlined form
2. **Adjust Timing Thresholds**: You can modify the minimum and maximum time thresholds in the validation function
3. **Add More Layers**: For higher security, you could integrate with external services like reCAPTCHA

## Maintenance Notes

- No third-party dependencies are required
- The solution is designed to be unobtrusive for legitimate users
- The code is well-documented for future maintenance