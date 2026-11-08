<template>
  <div class="app-wrapper">

    <aside class="sidebar">
      <div class="brand">
        <div class="logo-icon">P</div>
        <h2>People's Park</h2>
      </div>

      <nav class="nav-menu">
        <button 
          :class="{ active: currentTab === 'info' }" 
          @click="currentTab = 'info'"
        >
          <i class="icon">ℹ️</i> Info Hub
        </button>
        <button 
          :class="{ active: currentTab === 'booking' }" 
          @click="currentTab = 'booking'"
        >
          <i class="icon">🎟️</i> Book Facility
        </button>
        <button 
          :class="{ active: currentTab === 'group' }" 
          @click="currentTab = 'group'"
        >
          <i class="icon">👥</i> Group Monitor
        </button>
        <button 
          :class="{ active: currentTab === 'receptionist' }" 
          @click="currentTab = 'receptionist'"
        >
          <i class="icon">🛡️</i> Reception Desk
        </button>
        <button 
          :class="{ active: currentTab === 'lifeguard' }" 
          @click="currentTab = 'lifeguard'"
        >
          <i class="icon">🛟</i> Lifeguard Station
          <span v-if="hasActiveEmergency" class="badge-alert">!</span>
        </button>
      </nav>

      <div class="sidebar-footer">
        <div class="role-selector">
          <label>Active Role View:</label>
          <select v-model="activeRole">
            <option value="guest">Visitor / Guest</option>
            <option value="receptionist">Receptionist Staff</option>
            <option value="lifeguard">Lifeguard Staff</option>
          </select>
        </div>
      </div>
    </aside>

    <!-- Main Content Body -->
    <main class="main-content">
      <!-- Header Toolbar -->
      <header class="top-header">
        <div class="header-title">
          <h1>{{ currentTabTitle }}</h1>
          <p class="subtitle">San Carlos City Smart Park Operations & Monitoring</p>
        </div>

        <div class="user-profile">
          <div class="profile-info">
            <span class="user-name">{{ currentUser.name }}</span>
            <span class="user-id">ID: {{ currentUser.id }}</span>
          </div>
          <div class="avatar">{{ currentUser.initials }}</div>
        </div>
      </header>

      <!-- Dynamic Views -->
      <div class="content-body">
        <!-- 1. PUBLIC INFORMATION HUB -->
        <section v-if="currentTab === 'info'" class="tab-view">
          <div class="info-grid">
            <div class="card banner-card">
              <h3>Welcome to People's Park San Carlos City</h3>
              <p>Experience safe, modern, and accessible recreational spaces. Featuring real-time 
                visitor assistance, aquatic safety monitoring, and seamless digital reservations.</p>
            </div>

            <div class="card-row">
              <div class="card facility-card">
                <h4>🏞️ Around the Park</h4>
                <p>Open green spaces, promenade walks, and relaxation zones.</p>
                <span class="capacity">Capacity: 500 Visitors</span>
              </div>
              <div class="card facility-card">
                <h4>🏊 Swimming Pool</h4>
                <p>Olympic-sized public pool equipped with IoT safety tracking.</p>
                <span class="capacity">Capacity: 100 Swimmers</span>
              </div>
              <div class="card facility-card">
                <h4>🏛️ Pavilion</h4>
                <p>Event venue for private gatherings, community events, and seminars.</p>
                <span class="capacity">Capacity: 200 Guests</span>
              </div>
            </div>

            <div class="card announcement-card">
              <h4>📢 Latest Announcements & Events</h4>
              <ul>
                <li><strong>Park Maintenance:</strong> Pool cleaning scheduled for Monday 6:00 AM - 10:00 AM.</li>
                <li><strong>Weekend Cultural Show:</strong> Live performance at the Pavilion this Saturday at 5:00 PM.</li>
              </ul>
            </div>
          </div>
        </section>

        <!-- 2. RESERVATION & BOOKING PIPELINE -->
        <section v-if="currentTab === 'booking'" class="tab-view">
          <div class="booking-container">
            <div v-if="!completedReservation" class="card form-card">
              <h3>Facility Reservation & Wristband Registration</h3>
              
              <div class="form-group">
                <label>Select Facility:</label>
                <select v-model="bookingForm.facility">
                  <option value="Around the Park">Around the Park</option>
                  <option value="Swimming Pool">Swimming Pool</option>
                  <option value="Pavilion">Pavilion</option>
                </select>
              </div>

              <div class="form-group">
                <label>Number of Guests:</label>
                <input 
                  type="number" 
                  min="1" 
                  max="10" 
                  v-model.number="bookingForm.guestCount" 
                  @change="updateGuestFields"
                />
              </div>

              <div class="guest-list">
                <label>Guest Names (Wristband IDs are auto-assigned):</label>
                <div 
                  v-for="(guest, index) in bookingForm.guests" 
                  :key="index" 
                  class="guest-input-row"
                >
                  <input 
                    type="text" 
                    v-model="guest.name" 
                    :placeholder="'Guest #' + (index + 1) + ' Name'" 
                    required 
                  />
                  <span class="auto-id">Assigned Band: <strong>{{ guest.bandId }}</strong></span>
                </div>
              </div>

              <div class="form-group">
                <label>Payment Method:</label>
                <div class="radio-group">
                  <label><input type="radio" value="E-Wallet" v-model="bookingForm.paymentMethod" /> E-Wallet (GCash/PayMaya)</label>
                  <label><input type="radio" value="Cash" v-model="bookingForm.paymentMethod" /> Cash (Pay on Site)</label>
                </div>
              </div>

              <button class="primary-btn" @click="confirmBooking">Confirm Reservation & Generate Slip</button>
            </div>

            <!-- Online Reservation Slip -->
            <div v-else class="card slip-card">
              <div class="slip-header">
                <h3>🎟️ Online Reservation Slip</h3>
                <span class="status-badge">Confirmed</span>
              </div>
              <p><strong>Reservation Code:</strong> {{ completedReservation.code }}</p>
              <p><strong>Facility:</strong> {{ completedReservation.facility }}</p>
              <p><strong>Payment Method:</strong> {{ completedReservation.paymentMethod }}</p>
              <p><strong>Payment Status:</strong> {{ completedReservation.paymentMethod === 'E-Wallet' ? 'Paid (Digital Receipt Issued)' : 'Pending (Pay at Front Desk)' }}</p>

              <h4>Registered Wristband Tokens:</h4>
              <ul class="assigned-bands">
                <li v-for="guest in completedReservation.guests" :key="guest.bandId">
                  <span>{{ guest.name }}</span> — <strong>Band ID: {{ guest.bandId }}</strong>
                </li>
              </ul>
              <p class="note">⚠️ Please present this slip at the entrance receptionist desk to collect your physical IoT wristbands.</p>
              <button class="secondary-btn" @click="completedReservation = null">Book Another Facility</button>
            </div>
          </div>
        </section>

        <!-- 3. GUEST GROUP LOCATION MONITORING -->
        <section v-if="currentTab === 'group'" class="tab-view">
          <div class="card map-dashboard">
            <h3>Group Location & Safety Tracking</h3>
            <p class="subtitle">Monitor active wristbands assigned to your group reservation code.</p>

            <div class="map-container">
              <div class="park-map">
                <div class="zone pool-zone">Swimming Pool</div>
                <div class="zone pavilion-zone">Pavilion</div>
                <div class="zone park-zone">Around the Park</div>

                <!-- Live Map Pins -->
                <div 
                  v-for="guest in groupMembers" 
                  :key="guest.bandId"
                  class="map-pin"
                  :style="{ top: guest.posY + '%', left: guest.posX + '%' }"
                >
                  <div class="pin-bubble">{{ guest.bandId }}</div>
                </div>
              </div>
            </div>

            <div class="status-table">
              <h4>Linked Group Wristbands</h4>
              <table>
                <thead>
                  <tr>
                    <th>Band ID</th>
                    <th>Current Zone</th>
                    <th>Status</th>
                    <th>SpO2 Level</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="guest in groupMembers" :key="guest.bandId">
                    <td><strong>{{ guest.bandId }}</strong></td>
                    <td>{{ guest.zone }}</td>
                    <td>
                      <span :class="['status-pill', guest.status.toLowerCase()]">{{ guest.status }}</span>
                    </td>
                    <td>{{ guest.spO2 }}%</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </section>

        <!-- 4. RECEPTIONIST DASHBOARD -->
        <section v-if="currentTab === 'receptionist'" class="tab-view">
          <div class="reception-grid">
            <div class="card">
              <h3>Perimeter Integrity & Visitor Directory</h3>
              <p class="subtitle">Privacy Protocol: Guest names are kept administrative; tracking identifies spatial IDs only.</p>

              <table>
                <thead>
                  <tr>
                    <th>Band ID</th>
                    <th>Guest Name (Admin Only)</th>
                    <th>Current Location</th>
                    <th>Geofence Status</th>
                    <th>Action</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="visitor in allVisitors" :key="visitor.bandId">
                    <td><strong>{{ visitor.bandId }}</strong></td>
                    <td>{{ visitor.name }}</td>
                    <td>{{ visitor.zone }}</td>
                    <td>
                      <span :class="['status-pill', visitor.geofence.toLowerCase()]">
                        {{ visitor.geofence }}
                      </span>
                    </td>
                    <td>
                      <button 
                        v-if="visitor.geofence === 'Valid'" 
                        class="danger-btn-sm" 
                        @click="triggerGeofenceViolation(visitor)"
                      >
                        Simulate Exit
                      </button>
                      <span v-else class="text-flagged">Flagged Invalid</span>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </section>

        <!-- 5. LIFEGUARD SAFETY & EMERGENCY DASHBOARD -->
        <section v-if="currentTab === 'lifeguard'" class="tab-view">
          <div class="lifeguard-dashboard">
            <div v-if="hasActiveEmergency" class="alert-banner">
              🚨 EMERGENCY ALERT DETECTED! DISPATCH LIFEGUARD IMMEDIATELY! 🚨
            </div>

            <div class="card map-dashboard">
              <h3>Aquatic Safety & Live Biometric Response System</h3>
              <div class="map-container">
                <div class="park-map pool-focus">
                  <div class="zone pool-zone focused">Swimming Pool Zone</div>

                  <!-- Swimmer Pins -->
                  <div 
                    v-for="swimmer in poolSwimmers" 
                    :key="swimmer.bandId"
                    :class="['map-pin', swimmer.isEmergency ? 'emergency-pin' : '']"
                    :style="{ top: swimmer.posY + '%', left: swimmer.posX + '%' }"
                  >
                    <div class="pin-bubble">{{ swimmer.bandId }}</div>
                  </div>
                </div>
              </div>

              <!-- Simulator Panel for Testing Safety Features -->
              <div class="simulation-panel card">
                <h4>IoT Wristband Alert Simulator (Testing Controls)</h4>
                <div class="btn-group">
                  <button class="alert-btn" @click="simulateBiometricDrop">
                    Simulate SpO2 Drop (&lt;90%) / Drowning
                  </button>
                  <button class="alert-btn" @click="simulatePanicButton">
                    Simulate Physical SOS Button Press
                  </button>
                  <button class="reset-btn" @click="resetEmergencies">
                    Reset Emergency System
                  </button>
                </div>
              </div>

              <!-- Active Swimmer Biometrics -->
              <div class="status-table">
                <h4>Pool Area Biometric Feeds</h4>
                <table>
                  <thead>
                    <tr>
                      <th>Band ID</th>
                      <th>SpO2 Level</th>
                      <th>Heart Rate</th>
                      <th>Motion Status</th>
                      <th>Alert Status</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr 
                      v-for="swimmer in poolSwimmers" 
                      :key="swimmer.bandId"
                      :class="{ 'row-emergency': swimmer.isEmergency }"
                    >
                      <td><strong>{{ swimmer.bandId }}</strong></td>
                      <td :class="{ 'text-danger': swimmer.spO2 < 90 }">{{ swimmer.spO2 }}%</td>
                      <td>{{ swimmer.bpm }} BPM</td>
                      <td>{{ swimmer.motion }}</td>
                      <td>
                        <span v-if="swimmer.isEmergency" class="status-pill invalid">
                          🚨 {{ swimmer.alertType }}
                        </span>
                        <span v-else class="status-pill valid">Normal</span>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        </section>
      </div>
    </main>

    <!-- Right Side Dashboard Panel (Calendar & Active Users from UI Template) -->
    <aside class="right-panel">
      <!-- Calendar Widget -->
      <div class="card calendar-card">
        <div class="calendar-header">
          <h4>September 2026</h4>
          <div class="cal-nav"><span>&lt;</span> <span>&gt;</span></div>
        </div>
        <div class="calendar-grid">
          <span class="day-head">M</span><span class="day-head">T</span><span class="day-head">W</span><span class="day-head">T
          </span><span class="day-head">F</span><span class="day-head">S</span><span class="day-head">S</span>
          <span class="day-num">21</span><span class="day-num">22</span><span class="day-num active-day">23</span><span class="day-num">
            24</span><span class="day-num">25
          </span><span class="day-num">26</span><span class="day-num">27</span>
        </div>
      </div>

      <!-- Active On-Site Visitors (Online Users List from UI) -->
      <div class="card active-users-card">
        <h4>Active Staff & Guests</h4>
        <div class="user-list">
          <div v-for="user in activeOnlineUsers" :key="user.id" class="user-item">
            <div class="avatar-sm">{{ user.initials }}</div>
            <div class="user-details">
              <span class="name">{{ user.name }}</span>
              <span class="id-tag">ID: {{ user.id }}</span>
            </div>
            <span class="online-dot"></span>
          </div>
        </div>
      </div>
    </aside>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      currentTab: 'info',
      activeRole: 'guest',
      currentUser: {
        name: 'Christine Eva',
        id: '1094881999',
        initials: 'CE'
      },
      bookingForm: {
        facility: 'Around the Park',
        guestCount: 1,
        paymentMethod: 'E-Wallet',
        guests: [{ name: '', bandId: 'WB-101' }]
      },
      completedReservation: null,
      allVisitors: [
        { bandId: 'WB-101', name: 'Christine Eva', zone: 'Around the Park', geofence: 'Valid', posX: 65, posY: 55, spO2: 98, bpm: 72, motion: 'Normal', isEmergency: false, alertType: '' },
        { bandId: 'WB-102', name: 'Mark Lee', zone: 'Swimming Pool', geofence: 'Valid', posX: 25, posY: 30, spO2: 97, bpm: 85, motion: 'Swimming', isEmergency: false, alertType: '' },
        { bandId: 'WB-103', name: 'Jung Jaehyun', zone: 'Swimming Pool', geofence: 'Valid', posX: 35, posY: 40, spO2: 99, bpm: 78, motion: 'Swimming', isEmergency: false, alertType: '' },
        { bandId: 'WB-104', name: 'Kim Taeyeong', zone: 'Pavilion', geofence: 'Valid', posX: 75, posY: 25, spO2: 96, bpm: 70, motion: 'Stationary', isEmergency: false, alertType: '' }
      ],
      activeOnlineUsers: [
        { id: '1094882001', name: 'Maren Maureen', initials: 'MM' },
        { id: '1094672000', name: 'Jenniffer Jane', initials: 'JJ' },
        { id: '1094342003', name: 'Ryan Herwinds', initials: 'RH' },
        { id: '1094662002', name: 'Kierra Culhane', initials: 'KC' }
      ]
    };
  },
  computed: {
    currentTabTitle() {
      const titles = {
        info: 'Information Hub',
        booking: 'Reservation & Booking',
        group: 'Group Safety Tracking',
        receptionist: 'Receptionist Control Console',
        lifeguard: 'Lifeguard Emergency Station'
      };
      return titles[this.currentTab] || 'Dashboard';
    },
    groupMembers() {
      return this.allVisitors.filter(v => v.bandId === 'WB-101' || v.bandId === 'WB-102');
    },
    poolSwimmers() {
      return this.allVisitors.filter(v => v.zone === 'Swimming Pool');
    },
    hasActiveEmergency() {
      return this.allVisitors.some(v => v.isEmergency);
    }
  },
  methods: {
    updateGuestFields() {
      const count = Math.max(1, this.bookingForm.guestCount);
      const current = this.bookingForm.guests;
      if (count > current.length) {
        for (let i = current.length; i < count; i++) {
          current.push({
            name: '',
            bandId: `WB-${101 + i}`
          });
        }
      } else {
        this.bookingForm.guests = current.slice(0, count);
      }
    },
    confirmBooking() {
      const reservationCode = 'RES-' + Math.floor(100000 + Math.random() * 900000);
      this.completedReservation = {
        code: reservationCode,
        facility: this.bookingForm.facility,
        paymentMethod: this.bookingForm.paymentMethod,
        guests: JSON.parse(JSON.stringify(this.bookingForm.guests))
      };
    },
    triggerGeofenceViolation(visitor) {
      visitor.geofence = 'INVALID (Boundary Crossed)';
      alert(`⚠️ GEOFENCE ALERT: Wristband ${visitor.bandId} has exited the park without checking out! Ticket marked INVALID.`);
    },
    simulateBiometricDrop() {
      const swimmer = this.poolSwimmers[0];
      if (swimmer) {
        swimmer.spO2 = 84; // Drops below critical threshold 90%
        swimmer.motion = 'Erratic / Struggling';
        swimmer.isEmergency = true;
        swimmer.alertType = 'CRITICAL SpO2 DROP / DROWNING DETECTED';
        this.currentTab = 'lifeguard';
      }
    },
    simulatePanicButton() {
      const swimmer = this.poolSwimmers[1] || this.poolSwimmers[0];
      if (swimmer) {
        swimmer.isEmergency = true;
        swimmer.alertType = 'MANUAL SOS PANIC BUTTON PRESSED';
        this.currentTab = 'lifeguard';
      }
    },
    resetEmergencies() {
      this.allVisitors.forEach(v => {
        v.isEmergency = false;
        v.alertType = '';
        v.spO2 = 98;
        v.motion = 'Normal';
      });
    }
  }
};
</script>

<style scoped>
/* Main Layout structure following UI screenshot colors */
.app-wrapper {
  display: flex;
  height: 100vh;
  background-color: #f3f4f9;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
  color: #2d3748;
}

/* Sidebar Styling */
.sidebar {
  width: 240px;
  background-color: #5c59e8;
  color: #ffffff;
  display: flex;
  flex-direction: column;
  padding: 24px 16px;
}

.brand {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 32px;
}

.logo-icon {
  width: 36px;
  height: 36px;
  background: white;
  color: #5c59e8;
  font-weight: bold;
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
}

.nav-menu {
  display: flex;
  flex-direction: column;
  gap: 8px;
  flex: 1;
}

.nav-menu button {
  background: transparent;
  border: none;
  color: #d1d5db;
  padding: 12px 16px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 14px;
  cursor: pointer;
  text-align: left;
  transition: all 0.2s;
}

.nav-menu button.active, .nav-menu button:hover {
  background-color: rgba(255, 255, 255, 0.15);
  color: #ffffff;
  font-weight: 600;
}

.badge-alert {
  background-color: #ef4444;
  color: white;
  border-radius: 50%;
  padding: 2px 8px;
  font-size: 12px;
  margin-left: auto;
}

.role-selector select {
  width: 100%;
  padding: 8px;
  border-radius: 6px;
  border: none;
  margin-top: 6px;
}

/* Main Content Area */
.main-content {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  padding: 24px 32px;
}

.top-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.header-title h1 {
  font-size: 24px;
  color: #1e1b4b;
  margin: 0;
}

.subtitle {
  font-size: 13px;
  color: #6b7280;
  margin-top: 4px;
}

.user-profile {
  display: flex;
  align-items: center;
  gap: 12px;
}

.avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background-color: #c7d2fe;
  color: #3730a3;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
}

/* Cards & Grid Design */
.card {
  background: white;
  border-radius: 16px;
  padding: 20px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.03);
  margin-bottom: 16px;
}

.card-row {
  display: flex;
  gap: 16px;
}

.facility-card {
  flex: 1;
  background: #f9fafb;
  border: 1px solid #e5e7eb;
}

.capacity {
  font-size: 12px;
  color: #5c59e8;
  font-weight: 600;
}

/* Map Dashboard */
.map-container {
  width: 100%;
  height: 220px;
  background-color: #e0f2fe;
  border-radius: 12px;
  position: relative;
  margin: 16px 0;
  overflow: hidden;
  border: 2px solid #bae6fd;
}

.park-map {
  width: 100%;
  height: 100%;
  position: relative;
}

.zone {
  position: absolute;
  padding: 8px 12px;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 8px;
  font-size: 12px;
  font-weight: bold;
  color: #0369a1;
}

.pool-zone { top: 20%; left: 15%; width: 30%; height: 50%; background: #bae6fd; 
border: 1px dashed #0284c7; }
.pavilion-zone { top: 15%; right: 10%; }
.park-zone { bottom: 15%; right: 20%; }

.map-pin {
  position: absolute;
  transform: translate(-50%, -50%);
  transition: all 0.5s;
}

.pin-bubble {
  background: #5c59e8;
  color: white;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 10px;
  font-weight: bold;
  box-shadow: 0 2px 6px rgba(0,0,0,0.2);
}

.emergency-pin .pin-bubble {
  background: #dc2626;
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0% { transform: scale(1); }
  50% { transform: scale(1.2); }
  100% { transform: scale(1); }
}

/* Tables */
table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 12px;
}

th, td {
  text-align: left;
  padding: 10px;
  font-size: 13px;
  border-bottom: 1px solid #f3f4f6;
}

.status-pill {
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 11px;
  font-weight: 600;
  background: #d1fae5;
  color: #065f46;
}

.status-pill.invalid, .text-danger {
  background: #fee2e2;
  color: #991b1b;
}

/* Right Panel (UI Template Match) */
.right-panel {
  width: 280px;
  padding: 24px 16px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 8px;
  text-align: center;
  font-size: 12px;
  margin-top: 12px;
}

.active-day {
  background: #5c59e8;
  color: white;
  border-radius: 50%;
  padding: 4px;
}

.user-item {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-top: 12px;
}

.avatar-sm {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: #e0e7ff;
  color: #4338ca;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 11px;
  font-weight: bold;
}

.online-dot {
  width: 8px;
  height: 8px;
  background: #10b981;
  border-radius: 50%;
  margin-left: auto;
}

.alert-banner {
  background: #dc2626;
  color: white;
  padding: 12px;
  border-radius: 12px;
  font-weight: bold;
  text-align: center;
  margin-bottom: 16px;
  animation: pulse 1s infinite;
}

.btn-group {
  display: flex;
  gap: 8px;
  margin-top: 8px;
}

.alert-btn {
  background: #ef4444;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 12px;
}

.reset-btn {
  background: #6b7280;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 12px;
}

.primary-btn {
  background: #5c59e8;
  color: white;
  border: none;
  padding: 10px 16px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  margin-top: 12px;
}
</style>