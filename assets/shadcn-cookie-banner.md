### Modified version of kaizenics/shadcn-cookies
original repo: https://github.com/kaizenics/shadcn-cookies

```tsx
'use client';

import { CookieIcon } from 'lucide-react';
import Link from 'next/link';
import { useCallback, useEffect, useRef, useState } from 'react';

import { Button } from '@/components/ui/button';
import { cn } from '@/lib/utils';

type Variant = 'default' | 'small' | 'minimal';

interface CookieConsentProps {
  variant?: Variant;
  mode?: boolean;
  onAcceptCallback?: () => void;
  onDeclineCallback?: () => void;
  showDecline?: boolean;
  learnMoreUrl?: string;
  enableBackdrop?: boolean;
  disableBackgroundInteraction?: boolean;
  backdropClassName?: string;
}

export function CookieConsent({
  variant = 'default',
  mode = false,
  showDecline = true,
  learnMoreUrl = '/cookie-policy',
  onAcceptCallback,
  onDeclineCallback,
  enableBackdrop = true,
  disableBackgroundInteraction = false,
  backdropClassName = 'fixed inset-0 z-[199] bg-black/50 backdrop-blur-sm transition-opacity duration-700',
}: CookieConsentProps) {
  const [isOpen, setIsOpen] = useState(false);
  const [hide, setHide] = useState(false);
  const dialogRef = useRef<HTMLDivElement | null>(null);

  const COOKIE_NAME = '_bh4rl';

  const setCookie = (name: string, value: string, days = 365 * 10) => {
    const expires = new Date(Date.now() + days * 864e5).toUTCString();
    const cookieString = [
      `${encodeURIComponent(name)}=${encodeURIComponent(value)}`,
      `expires=${expires}`,
      'path=/',
      'SameSite=Lax',
    ];

    if (typeof window !== 'undefined' && location.protocol === 'https:') {
      cookieString.push('Secure');
    }

    if (typeof document !== 'undefined') {
      // biome-ignore lint/suspicious/noDocumentCookie: <needed>
      document.cookie = cookieString.join('; ');
    }
  };

  const hasCookieConsent = useCallback(() => {
    return typeof document !== 'undefined' && document.cookie.includes(`${COOKIE_NAME}=true`);
  }, []);

  const resetBodyStyles = useCallback(() => {
    if (typeof document !== 'undefined') {
      document.body.style.overflow = '';
      document.body.style.pointerEvents = '';
    }
  }, []);

  const accept = () => {
    setIsOpen(false);
    setCookie(COOKIE_NAME, 'true');
    onAcceptCallback?.();

    resetBodyStyles();

    setTimeout(() => setHide(true), 700);
  };

  const decline = () => {
    setIsOpen(false);
    onDeclineCallback?.();

    resetBodyStyles();

    setTimeout(() => setHide(true), 700);
  };

  useEffect(() => {
    const shouldShow = !hasCookieConsent() || mode;

    if (shouldShow) {
      setIsOpen(true);
      if (typeof document !== 'undefined') {
        document.body.style.overflow = 'hidden';
      }
    } else {
      setHide(true);
    }

    return () => {
      resetBodyStyles();
    };
  }, [hasCookieConsent, mode, resetBodyStyles]);

  useEffect(() => {
    if (!isOpen || !disableBackgroundInteraction) return;

    if (typeof document !== 'undefined') {
      document.body.style.pointerEvents = 'none';
    }

    return () => {
      if (typeof document !== 'undefined') {
        document.body.style.pointerEvents = '';
      }
    };
  }, [isOpen, disableBackgroundInteraction]);

  useEffect(() => {
    if (!isOpen || !dialogRef.current) return;

    const focusableSelectors = [
      'a[href]',
      'button:not([disabled])',
      'textarea:not([disabled])',
      'input:not([disabled])',
      'select:not([disabled])',
      '[tabindex]:not([tabindex="-1"])',
    ];
    const elements = dialogRef.current.querySelectorAll<HTMLElement>(focusableSelectors.join(','));
    const first = elements[0];
    const last = elements[elements.length - 1];

    const handleKeyDown = (e: KeyboardEvent) => {
      if (e.key !== 'Tab') return;
      if (elements.length === 0) return;

      if (e.shiftKey) {
        if (document.activeElement === first) {
          e.preventDefault();
          last.focus();
        }
      } else {
        if (document.activeElement === last) {
          e.preventDefault();
          first.focus();
        }
      }
    };

    first?.focus();
    document.addEventListener('keydown', handleKeyDown);
    return () => document.removeEventListener('keydown', handleKeyDown);
  }, [isOpen]);

  if (hide) return null;

  const wrapperClasses = cn(
    'fixed z-[200] bottom-0 left-0 right-0 p-4 sm:p-0 sm:left-4 sm:bottom-4 w-full sm:max-w-md transition-all duration-700',
    !isOpen && 'translate-y-8 opacity-0 pointer-events-none',
    'pointer-events-auto',
  );

  const baseBoxClasses = 'dark:bg-card bg-background rounded-lg border border-border shadow-lg';

  const Overlay = enableBackdrop ? (
    <div
      className={backdropClassName}
      aria-hidden="true"
    />
  ) : null;

  const commonDialogProps = {
    role: 'dialog',
    'aria-modal': true,
    'aria-label': 'Cookie Consent',
    tabIndex: -1,
    ref: dialogRef,
  };

  const renderDefault = () => (
    <>
      {Overlay}
      <div
        className={wrapperClasses}
        {...commonDialogProps}
      >
        <div className={cn(baseBoxClasses)}>
          <div className="grid gap-2">
            <header className="border-b border-border h-12 sm:h-14 flex items-center justify-between px-4">
              <h1 className="text-base sm:text-lg font-medium">We use cookies</h1>
              <CookieIcon className="h-4 w-4 sm:h-5 sm:w-5" />
            </header>
            <div className="px-4 py-2 text-sm">
              <p>
                We use cookies to ensure you get the best experience on our website. For more
                information, see our cookie policy.
              </p>
              <p className="mt-2 text-xs">
                By using our website <span className="font-medium text-primary">Accept</span>, you
                agree to our use of cookies.
              </p>
              {learnMoreUrl && (
                <Link
                  href={learnMoreUrl}
                  target="_blank"
                  rel="noopener noreferrer"
                  className="text-xs text-primary"
                >
                  Learn more
                </Link>
              )}
            </div>
            <footer className="flex flex-col sm:flex-row gap-2 px-4 py-3 border-t border-border justify-end">
              <Button
                onClick={accept}
                className="!w-max !text-white"
              >
                Accept
              </Button>
              {showDecline && (
                <Button
                  onClick={decline}
                  variant="secondary"
                  className="w-full"
                >
                  Decline
                </Button>
              )}
            </footer>
          </div>
        </div>
      </div>
    </>
  );

  const renderSmall = () => (
    <>
      {Overlay}
      <div
        className={wrapperClasses}
        {...commonDialogProps}
      >
        <div className={cn(baseBoxClasses)}>
          <div className="flex items-center justify-between px-4 py-3">
            <h2 className="text-sm sm:text-base font-medium">We use cookies</h2>
            <CookieIcon className="h-4 w-4" />
          </div>
          <div className="px-4 pb-2 text-xs text-muted-foreground">
            We use cookies to improve your experience.
            {learnMoreUrl && (
              <div className="mt-1">
                <Link
                  href={learnMoreUrl}
                  className="text-primary text-xs"
                  target="_blank"
                  rel="noopener noreferrer"
                >
                  Learn more
                </Link>
              </div>
            )}
          </div>
          <div className="flex flex-col sm:flex-row gap-2 px-4 py-3 border-t border-border justify-end">
            <Button
              onClick={accept}
              className="w-full text-xs text-white"
            >
              Accept
            </Button>
            {showDecline && (
              <Button
                onClick={decline}
                variant="outline"
                className="w-full text-xs text-white"
              >
                Decline
              </Button>
            )}
          </div>
        </div>
      </div>
    </>
  );

  const renderMinimal = () => (
    <>
      {Overlay}
      <div
        className={wrapperClasses}
        {...commonDialogProps}
      >
        <div className={cn(baseBoxClasses, 'sm:max-w-[300px]')}>
          <div className="flex items-center justify-between px-3 py-2 border-b border-border">
            <div className="flex items-center gap-2">
              <CookieIcon className="h-4 w-4" />
              <span className="text-sm font-medium">Cookie Notice</span>
            </div>
          </div>
          <div className="px-3 py-2 text-xs text-muted-foreground">
            <p>We use cookies to enhance your experience.</p>
            {learnMoreUrl && (
              <Link
                href={learnMoreUrl}
                className="underline text-xs block mt-1"
                target="_blank"
                rel="noopener noreferrer"
              >
                Learn more
              </Link>
            )}
            <div className="flex flex-col sm:flex-row gap-2 mt-3">
              <Button
                onClick={accept}
                size="sm"
                className="w-full h-7 text-xs px-3 text-white"
              >
                Accept
              </Button>
              {showDecline && (
                <Button
                  onClick={decline}
                  size="sm"
                  variant="ghost"
                  className="w-full h-7 text-xs px-3 text-white"
                >
                  Decline
                </Button>
              )}
            </div>
          </div>
        </div>
      </div>
    </>
  );

  if (variant === 'minimal') return renderMinimal();
  if (variant === 'small') return renderSmall();
  return renderDefault();
}

```

