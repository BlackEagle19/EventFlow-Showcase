# EventFlow - Intelligent B2B SaaS Event Management Platform

> **IMPORTANT DISCLAIMER**
> This is a commercial B2B SaaS product developed under **Vectorium Technology**. The source code is private and proprietary due to Non-Disclosure Agreements (NDA). The content below serves as a technical overview and architectural showcase for portfolio purposes.

## Project Overview

**EventFlow** is a comprehensive, enterprise-grade generic B2B SaaS platform designed to streamline event management, appointment scheduling, and venue logic for businesses. It provides a unified ecosystem for managing complex calendar availability, service provider allocations, financial reporting, and customer interactions through a high-performance, responsive web interface.

Built with scalability in mind, EventFlow leverages a modern architecture to handle real-time data synchronization, multi-tenant resource management, and predictive analytics.

---

## Technology Stack

The platform allows for high concurrency, real-time updates, and complex data visualization.

### Frontend Engineering
*   **Core Framework:** Next.js 15 (React 18) - Utilizing Server Side Rendering (SSR) and App Router for optimized performance.
*   **Language & Logic:** Modern JavaScript (ES6+), optimized for strict type safety and modularity.
*   **UI System:** Chakra UI & Radix UI primitives, styled with Tailwind CSS for a custom, adaptive design system.
*   **State Management:** React Hook Form for complex form handling, SWR for stale-while-revalidate data fetching.
*   **Visualization:** Recharts for dynamic analytics boards, FullCalendar for interactive scheduling.
*   **Maps & Geo:** Leaflet and React-Leaflet for location-based service management.

### Backend Infrastructure
*   **Core Framework:** Python 3.12 / Django 5.2 - Robust, secure, and scalable high-level web framework.
*   **API Architecture:** Django REST Framework (DRF) following RESTful principles with nested routing.
*   **Real-time Communication:** Django Channels & Redis for WebSocket handling (live notifications and updates).
*   **Task Queue:** Celery & Redis for asynchronous background processing (reports, emails, periodic tasks).
*   **Data Science:** Pandas & Prophet for sales forecasting and trend analysis.
*   **Database:** PostgreSQL (Production) / SQLite (Dev) with spatial extensions (PostGIS).

### DevOps & Tools
*   **Containerization:** Docker & Docker Compose for consistent development and deployment environments.
*   **Server:** Gunicorn behind Nginx reverse proxy.
*   **Monitoring:** Sentry for real-time error tracking and performance monitoring.
*   **CI/CD:** Automated pipelines for testing and deployment.

---

## Key Features

*   **Dynamic Dashboard:** Real-time financial summaries, appointment trends, and occupancy rates visualized with interactive charts.
*   **Advanced Scheduling Engine:** Conflict-free Booking System handling Holidays, Special Days, and custom Business Hours.
*   **Role-Based Access Control (RBAC):** Granular permission scopes for Business Owners, Staff, and End-Customers.
*   **Real-time Notifications:** WebSocket-driven alerts for new appointments, cancellations, and status changes.
*   **Geospatial Logic:** Location verification for mobile services and venue tracking.
*   **Predictive Analytics:** AI-powered demand forecasting using Prophet library.
*   **Financial Suite:** Automated invoicing, expense tracking, and revenue reporting exportable to PDF/Excel.

---

## Architecture & Methodology

The application follows a **Microservices-ready Monolithic Architecture**. The backend is modularized into distinct domains (`appointments`, `businesses`, `financial`, `mobile_services`) to ensure separation of concerns and maintainability.

*   **Component-Driven Development:** The frontend utilizes atomic design principles, ensuring UI consistency across the platform.
*   **Optimized Rendering:** Next.js Hybrid rendering (SSG + SSR) is used to balance SEO needs with dynamic user content.
*   **Custom Hooks:** Complex logic is extracted into reusable hooks (e.g., `useAuthRequired`, `useBusinessNotifications`) to keep components pure.
*   **Service Layer Pattern:** Backend logic is encapsulated in a Service Layer, keeping Views and Models lean.

---

## Engineering Highlights

### Complex Availability Logic (Backend)
The following snippet demonstrates the calculation of available time slots for a specific venue. This robust algorithm accounts for:
1.  Variable Venue Operating Hours.
2.  Existing Reservations (both manual and system-generated).
3.  Minimum Reservation Lead Time.
4.  Dynamic Duration Steps (based on rental configuration).

```python
# EventFlowBackend/appointments/views.py

def calculate_available_slots(self, venue, selected_date):
    """
    Calculates available time slots for a venue based on complex business rules.
    """
    # 1. Determine Working Hours
    working_hours = venue.get_working_hours_for_date(selected_date)
    if not working_hours:
        return []

    # 2. Check Minimum Lead Time (prevent last-minute bookings)
    now = timezone.now()
    min_lead_time_seconds = venue.min_time_before_reservation * 60
    
    # 3. Retrieve Existing Bookings
    existing_appointments = Appointment.objects.filter(
        venue=venue,
        date=selected_date
    ).exclude(status='cancelled')

    booked_times = set(app.time for app in existing_appointments)

    # 4. Generate Slots
    slots = []
    current_time = working_hours['start']
    end_time = working_hours['end']
    duration_minutes = venue.rental_duration * 60

    while current_time < end_time:
        # Check if slot is already booked
        if current_time not in booked_times:
            # Check time validity against current time + lead time
            slot_datetime = datetime.combine(selected_date, current_time)
            time_diff = (slot_datetime - now).total_seconds()
            
            if time_diff >= min_lead_time_seconds:
                slots.append(current_time)
        
        # Increment by duration
        current_time = self.add_minutes(current_time, duration_minutes)

    return slots
```

### Protected Route Hook (Frontend)
A custom React Hook designed to handle authentication state and guard protected actions across the application.

```javascript
// eventflow-web/hooks/useAuthRequired.js

const useAuthRequired = () => {
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [isCheckingAuth, setIsCheckingAuth] = useState(true);

  useEffect(() => {
    const checkAuthStatus = async () => {
      try {
        const isAuth = await CustomerAuthService.isAuthenticated();
        setIsLoggedIn(isAuth);
      } catch (error) {
        setIsLoggedIn(false);
      } finally {
        setIsCheckingAuth(false);
      }
    };
    checkAuthStatus();
  }, []);

  const requireAuth = (callback, config = {}) => {
    if (isCheckingAuth) return;
    
    if (!isLoggedIn) {
      // Trigger global auth modal with custom context
      setShowAuthModal(true, {
        title: config.title || 'Login Required',
        description: config.description
      });
      return false;
    }
    
    // Execute protected action
    if (callback) callback();
    return true;
  };

  return { isLoggedIn, requireAuth };
};
```

---

## Visuals

![SaaS Dashboard](./assets/dashboard.png)
*Figure 1: Main Dashboard Visualization (Placeholder)*

![Analytics View](./assets/analytics.png)
*Figure 2: Financial Analytics & Trends (Placeholder)*

---

## Team

**Developed by the Engineering Team at Vectorium Technology**

* **Mehmet Furkan Güneş** - Co-Founder & Full Stack Engineer
    * *Focus:* Mobile Architecture (React Native), UI/UX Design, Client-Side Logic.
* **Nihal Kemer** - Co-Founder & Full Stack Engineer
    * *Focus:* System Architecture, API Development (Django), Database Optimization.
