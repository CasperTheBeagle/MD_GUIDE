# 📊 คู่มือการติดตามและตรวจสอบระบบ

## 🎯 วัตถุประสงค์
เอกสารนี้ให้คำแนะนำในการติดตามและตรวจสอบระบบ React applications เพื่อให้มั่นใจว่าระบบทำงานได้อย่างมีเสถียรภาพและมีประสิทธิภาพ

## 📋 สารบัญ
- [การติดตามประสิทธิภาพ](#-การติดตามประสิทธิภาพ)
- [การติดตามความปลอดภัย](#-การติดตามความปลอดภัย)
- [การติดตามการใช้งาน](#-การติดตามการใช้งาน)
- [การติดตามข้อผิดพลาด](#-การติดตามข้อผิดพลาด)
- [เครื่องมือที่ใช้](#-เครื่องมือที่ใช้)

## 📈 การติดตามประสิทธิภาพ

### 1. ตัวชี้วัดประสิทธิภาพ (KPIs)
- **Response Time**: เวลาตอบสนองเฉลี่ย
- **Throughput**: จำนวนคำขอต่อวินาที
- **CPU Usage**: การใช้งาน CPU
- **Memory Usage**: การใช้งานหน่วยความจำ
- **Database Performance**: ประสิทธิภาพฐานข้อมูล
- **Network Latency**: ความหน่วงของเครือข่าย

### 2. การติดตามแบบ Real-time
```typescript
// ✅ ดี - Performance monitoring setup
import { performance } from 'perf_hooks';

class PerformanceMonitor {
  private metrics: Map<string, number[]> = new Map();

  startTimer(name: string): number {
    return performance.now();
  }

  endTimer(name: string, startTime: number): void {
    const duration = performance.now() - startTime;
    if (!this.metrics.has(name)) {
      this.metrics.set(name, []);
    }
    this.metrics.get(name)!.push(duration);
  }

  getAverage(name: string): number {
    const values = this.metrics.get(name) || [];
    return values.reduce((sum, val) => sum + val, 0) / values.length;
  }

  getMetrics(): Record<string, any> {
    const result: Record<string, any> = {};
    for (const [name, values] of this.metrics.entries()) {
      result[name] = {
        count: values.length,
        average: this.getAverage(name),
        min: Math.min(...values),
        max: Math.max(...values),
      };
    }
    return result;
  }
}

// การใช้งาน
const monitor = new PerformanceMonitor();

// ติดตาม API response time
app.use((req, res, next) => {
  const startTime = monitor.startTimer('api_response');
  res.on('finish', () => {
    monitor.endTimer('api_response', startTime);
  });
  next();
});
```

### 3. การติดตาม Frontend Performance
```typescript
// ✅ ดี - Frontend performance monitoring
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

class FrontendPerformanceMonitor {
  init(): void {
    // Core Web Vitals
    getCLS(this.logMetric);
    getFID(this.logMetric);
    getFCP(this.logMetric);
    getLCP(this.logMetric);
    getTTFB(this.logMetric);

    // Custom metrics
    this.trackPageLoad();
    this.trackUserInteractions();
    this.trackResourceLoading();
  }

  private logMetric = (metric: any): void => {
    console.log(metric);
    // ส่งข้อมูลไปยัง monitoring service
    this.sendToMonitoringService(metric);
  };

  private trackPageLoad(): void {
    window.addEventListener('load', () => {
      const navigation = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
      const metrics = {
        domContentLoaded: navigation.domContentLoadedEventEnd - navigation.domContentLoadedEventStart,
        loadComplete: navigation.loadEventEnd - navigation.loadEventStart,
        firstPaint: performance.getEntriesByType('paint')[0]?.startTime,
        firstContentfulPaint: performance.getEntriesByType('paint')[1]?.startTime,
      };
      this.sendToMonitoringService(metrics);
    });
  }

  private trackUserInteractions(): void {
    ['click', 'scroll', 'keydown'].forEach(event => {
      document.addEventListener(event, () => {
        const timestamp = performance.now();
        this.sendToMonitoringService({ event, timestamp });
      }, { passive: true });
    });
  }

  private trackResourceLoading(): void {
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        if (entry.entryType === 'resource') {
          const resource = entry as PerformanceResourceTiming;
          this.sendToMonitoringService({
            name: resource.name,
            duration: resource.duration,
            size: resource.transferSize,
            type: resource.initiatorType,
          });
        }
      }
    });
    observer.observe({ entryTypes: ['resource'] });
  }

  private sendToMonitoringService(data: any): void {
    // ส่งข้อมูลไปยัง monitoring service
    fetch('/api/metrics', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    }).catch(console.error);
  }
}

// การใช้งาน
const frontendMonitor = new FrontendPerformanceMonitor();
frontendMonitor.init();
```

## 🔒 การติดตามความปลอดภัย

### 1. การติดตามการเข้าสู่ระบบ
```typescript
// ✅ ดี - Security monitoring
class SecurityMonitor {
  private suspiciousIPs = new Set<string>();
  private failedAttempts = new Map<string, number>();

  trackLoginAttempt(email: string, ip: string, success: boolean, userAgent: string): void {
    const event = {
      timestamp: new Date().toISOString(),
      email: this.maskEmail(email),
      ip,
      success,
      userAgent,
      location: this.getIPLocation(ip),
    };

    this.logSecurityEvent('LOGIN_ATTEMPT', event);

    if (!success) {
      this.handleFailedLogin(ip);
    }
  }

  trackPasswordChange(userId: string, ip: string): void {
    const event = {
      timestamp: new Date().toISOString(),
      userId,
      ip,
      location: this.getIPLocation(ip),
    };

    this.logSecurityEvent('PASSWORD_CHANGE', event);
  }

  trackSuspiciousActivity(description: string, ip: string, userId?: string): void {
    const event = {
      timestamp: new Date().toISOString(),
      description,
      ip,
      userId,
      location: this.getIPLocation(ip),
    };

    this.logSecurityEvent('SUSPICIOUS_ACTIVITY', event);
    this.alertSecurityTeam(event);
  }

  private handleFailedLogin(ip: string): void {
    const attempts = this.failedAttempts.get(ip) || 0;
    this.failedAttempts.set(ip, attempts + 1);

    if (attempts + 1 >= 5) {
      this.suspiciousIPs.add(ip);
      this.trackSuspiciousActivity('Multiple failed login attempts', ip);
    }
  }

  private logSecurityEvent(type: string, data: any): void {
    console.log(`SECURITY: ${type}`, data);
    // ส่งไปยัง SIEM system
    this.sendToSIEM(type, data);
  }

  private maskEmail(email: string): string {
    const [username, domain] = email.split('@');
    return `${username.slice(0, 2)}***@${domain}`;
  }

  private getIPLocation(ip: string): string {
    // ใช้ GeoIP service
    return 'Unknown'; // ตัวอย่าง
  }

  private sendToSIEM(type: string, data: any): void {
    // ส่งข้อมูลไปยัง SIEM system
  }

  private alertSecurityTeam(event: any): void {
    // แจ้งเตือนทีมความปลอดภัย
  }
}
```

### 2. การติดตามการเข้าถึงข้อมูล
```typescript
// ✅ ดี - Data access monitoring
class DataAccessMonitor {
  trackDataAccess(userId: string, resource: string, action: string, ip: string): void {
    const event = {
      timestamp: new Date().toISOString(),
      userId,
      resource,
      action,
      ip,
      success: true,
    };

    this.logDataAccess(event);
  }

  trackDataAccessFailure(userId: string, resource: string, action: string, ip: string, reason: string): void {
    const event = {
      timestamp: new Date().toISOString(),
      userId,
      resource,
      action,
      ip,
      success: false,
      reason,
    };

    this.logDataAccess(event);
    this.trackSuspiciousAccess(event);
  }

  private logDataAccess(event: any): void {
    console.log('DATA_ACCESS:', event);
    // บันทึกไปยัง audit log
    this.writeToAuditLog(event);
  }

  private trackSuspiciousAccess(event: any): void {
    // ติดตามการเข้าถึงที่น่าสงสัย
    if (this.isSuspiciousAccess(event)) {
      this.alertSecurityTeam(event);
    }
  }

  private isSuspiciousAccess(event: any): boolean {
    // ตรวจสอบว่าการเข้าถึงน่าสงสัยหรือไม่
    return false;
  }

  private writeToAuditLog(event: any): void {
    // เขียนไปยัง audit log
  }

  private alertSecurityTeam(event: any): void {
    // แจ้งเตือนทีมความปลอดภัย
  }
}
```

## 👥 การติดตามการใช้งาน

### 1. การติดตามพฤติกรรมผู้ใช้
```typescript
// ✅ ดี - User behavior tracking
class UserBehaviorMonitor {
  trackPageView(page: string, userId?: string): void {
    const event = {
      type: 'page_view',
      page,
      userId,
      timestamp: new Date().toISOString(),
      referrer: document.referrer,
      userAgent: navigator.userAgent,
    };

    this.sendToAnalytics(event);
  }

  trackUserAction(action: string, properties: Record<string, any>, userId?: string): void {
    const event = {
      type: 'user_action',
      action,
      properties,
      userId,
      timestamp: new Date().toISOString(),
    };

    this.sendToAnalytics(event);
  }

  trackSessionStart(userId?: string): void {
    const event = {
      type: 'session_start',
      userId,
      timestamp: new Date().toISOString(),
      sessionId: this.generateSessionId(),
    };

    this.sendToAnalytics(event);
  }

  trackSessionEnd(sessionId: string, duration: number): void {
    const event = {
      type: 'session_end',
      sessionId,
      duration,
      timestamp: new Date().toISOString(),
    };

    this.sendToAnalytics(event);
  }

  private sendToAnalytics(event: any): void {
    // ส่งข้อมูลไปยัง analytics service
    fetch('/api/analytics', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(event),
    }).catch(console.error);
  }

  private generateSessionId(): string {
    return Math.random().toString(36).substr(2, 9);
  }
}
```

### 2. การติดตาม Conversion
```typescript
// ✅ ดี - Conversion tracking
class ConversionMonitor {
  trackConversion(type: string, value: number, currency: string, userId?: string): void {
    const event = {
      type: 'conversion',
      conversionType: type,
      value,
      currency,
      userId,
      timestamp: new Date().toISOString(),
    };

    this.sendToAnalytics(event);
  }

  trackFunnelStep(step: string, userId?: string): void {
    const event = {
      type: 'funnel_step',
      step,
      userId,
      timestamp: new Date().toISOString(),
    };

    this.sendToAnalytics(event);
  }

  trackFeatureUsage(feature: string, userId?: string): void {
    const event = {
      type: 'feature_usage',
      feature,
      userId,
      timestamp: new Date().toISOString(),
    };

    this.sendToAnalytics(event);
  }

  private sendToAnalytics(event: any): void {
    // ส่งข้อมูลไปยัง analytics service
  }
}
```

## 🚨 การติดตามข้อผิดพลาด

### 1. Error Tracking
```typescript
// ✅ ดี - Error tracking system
class ErrorTracker {
  private errors: Map<string, number> = new Map();

  trackError(error: Error, context?: Record<string, any>): void {
    const errorInfo = {
      message: error.message,
      stack: error.stack,
      timestamp: new Date().toISOString(),
      url: window.location.href,
      userAgent: navigator.userAgent,
      context: context || {},
    };

    this.logError(errorInfo);
    this.incrementErrorCount(error.message);
  }

  trackUnhandledRejection(event: PromiseRejectionEvent): void {
    const errorInfo = {
      message: 'Unhandled Promise Rejection',
      reason: event.reason,
      timestamp: new Date().toISOString(),
      url: window.location.href,
      userAgent: navigator.userAgent,
    };

    this.logError(errorInfo);
  }

  private logError(errorInfo: any): void {
    console.error('ERROR:', errorInfo);
    // ส่งไปยัง error tracking service
    this.sendToErrorService(errorInfo);
  }

  private incrementErrorCount(message: string): void {
    const count = this.errors.get(message) || 0;
    this.errors.set(message, count + 1);

    // แจ้งเตือนถ้ามีข้อผิดพลาดซ้ำๆ
    if (count >= 10) {
      this.alertDevelopmentTeam(message);
    }
  }

  private sendToErrorService(errorInfo: any): void {
    // ส่งไปยัง error tracking service เช่น Sentry
  }

  private alertDevelopmentTeam(message: string): void {
    // แจ้งเตือนทีมพัฒนา
  }
}

// การใช้งาน
const errorTracker = new ErrorTracker();

// ติดตามข้อผิดพลาดทั่วทั้งแอปพลิเคชัน
window.addEventListener('error', (event) => {
  errorTracker.trackError(event.error);
});

window.addEventListener('unhandledrejection', (event) => {
  errorTracker.trackUnhandledRejection(event);
});

// ใน React Error Boundary
class ErrorBoundary extends React.Component<{children: React.ReactNode}, {hasError: boolean}> {
  constructor(props: any) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): {hasError: boolean} {
    return { hasError: true };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo): void {
    errorTracker.trackError(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong.</h1>;
    }

    return this.props.children;
  }
}
```

## 🛠️ เครื่องมือที่ใช้

### 1. Application Performance Monitoring (APM)
- **New Relic**: ติดตามประสิทธิภาพแอปพลิเคชัน
- **Datadog**: ติดตามและวิเคราะห์ประสิทธิภาพ
- **AppDynamics**: ติดตามประสิทธิภาพและการใช้งาน
- **Dynatrace**: ติดตามประสิทธิภาพแบบ end-to-end

### 2. Error Tracking
- **Sentry**: ติดตามข้อผิดพลาดและประสิทธิภาพ
- **Bugsnag**: ติดตามข้อผิดพลาดและการทำงาน
- **Rollbar**: ติดตามข้อผิดพลาดแบบ real-time
- **Airbrake**: ติดตามข้อผิดพลาดและประสิทธิภาพ

### 3. Analytics
- **Google Analytics**: วิเคราะห์การใช้งานเว็บไซต์
- **Mixpanel**: วิเคราะห์พฤติกรรมผู้ใช้
- **Amplitude**: วิเคราะห์ผลิตภัณฑ์และการใช้งาน
- **Segment**: รวบรวมข้อมูล analytics

### 4. Infrastructure Monitoring
- **Prometheus**: ติดตามระบบและแอปพลิเคชัน
- **Grafana**: แสดงภาพข้อมูลการติดตาม
- **ELK Stack**: วิเคราะห์ logs และข้อมูล
- **Splunk**: วิเคราะห์ข้อมูลและ logs

## 📊 Dashboard และ Reports

### 1. Performance Dashboard
- Response time trends
- Throughput metrics
- Error rates
- Resource utilization
- Database performance

### 2. Security Dashboard
- Login attempts
- Failed authentications
- Suspicious activities
- Security incidents
- Compliance status

### 3. User Analytics Dashboard
- Active users
- Page views
- User sessions
- Conversion rates
- Feature usage

### 4. Error Dashboard
- Error rates by type
- Error trends
- Top errors
- Error impact
- Resolution status

## 🚨 Alerts และ Notifications

### 1. Alert Rules
- **Critical**: ระบบล้มเหลวทั้งหมด
- **High**: ประสิทธิภาพต่ำกว่าเกณฑ์
- **Medium**: อัตราข้อผิดพลาดสูง
- **Low**: การใช้งานทรัพยากรสูง

### 2. Notification Channels
- **Email**: แจ้งเตือนทางอีเมล
- **SMS**: แจ้งเตือนทาง SMS
- **Slack**: แจ้งเตือนใน Slack
- **PagerDuty**: แจ้งเตือนฉุกเฉิน

---

*เอกสารนี้อัปเดตล่าสุด: 20 มกราคม 2024*  
*เวอร์ชัน: 1.0*
