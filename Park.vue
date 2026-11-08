<template>
  <div class="app-viewport">
    <div class="app-container">
      <!-- LEFT SIDEBAR -->
      <aside class="sidebar">
        <div class="brand">
          <div class="brand-icon">
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
              <path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/>
            </svg>
          </div>
          <div class="brand-text">
            <h2>People's Park</h2>
            <span>San Carlos City</span>
          </div>
        </div>

        <nav class="nav-menu">
          <button 
            v-for="item in navItems" 
            :key="item.id"
            :class="['nav-btn', { active: activeTab === item.id }]"
            @click="activeTab = item.id"
          >
            <span class="nav-icon" v-html="item.icon"></span>
            <span class="nav-label">{{ item.label }}</span>
          </button>
        </nav>

        <div class="sidebar-footer">
          <button 
            v-for="item in secondaryNavItems" 
            :key="item.id"
            :class="['nav-btn', { active: activeTab === item.id }]"
            @click="activeTab = item.id"
          >
            <span class="nav-icon" v-html="item.icon"></span>
            <span class="nav-label">{{ item.label }}</span>
          </button>
        </div>
      </aside>

      <!-- MAIN CONTENT AREA -->
      <main class="main-content">
        <!-- TOP HEADER -->
        <header class="top-header">
          <div class="header-title">
            <h1>{{ activeTabTitle }}</h1>
            <p>Integrated Smart Park & Safety Monitoring Portal</p>
          </div>

          <div class="header-actions">
            <!-- FUNCTIONAL SEARCH BAR WITH DROPDOWN -->
            <div class="search-wrapper">
              <div :class="['search-bar', { 'is-focused': isSearchFocused || searchQuery }]">
                <svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <circle cx="11" cy="11" r="8"/><line x1="21" y1="21" x2="16.65" y2="16.65"/>
                </svg>
                <input 
                  type="text" 
                  placeholder="Search wristband ID, name, zone, or ticket..." 
                  v-model="searchQuery"
                  @focus="isSearchFocused = true"
                  @blur="handleSearchBlur"
                />
                <button v-if="searchQuery" class="clear-search" @click="searchQuery = ''">&times;</button>
              </div>

              <!-- LIVE SEARCH RESULTS DROPDOWN -->
              <div v-if="isSearchFocused && searchQuery.trim()" class="search-results-dropdown">
                <div v-if="searchResults.length === 0" class="no-results">
                  No matching wristbands, visitors, or zones found for "{{ searchQuery }}"
                </div>

                <div v-else class="results-list">
                  <div class="results-group-title">Live Active Visitors ({{ searchResults.length }})</div>
                  <div 
                    v-for="res in searchResults" 
                    :key="res.wristbandId" 
                    class="search-result-item"
                    @mousedown="selectSearchResult(res)"
                  >
                    <div class="res-info">
                      <span class="badge-wb">{{ res.wristbandId }}</span>
                      <strong class="res-name">{{ res.name }}</strong>
                    </div>
                    <div class="res-meta">
                      <span class="res-zone">{{ res.zone }}</span>
                      <span :class="['status-pill-sm', res.inDistress ? 'pill-danger' : res.outOfBounds ? 'pill-warning' : 'pill-ok']">
                        {{ res.inDistress ? 'Distress Alert' : res.outOfBounds ? 'Invalid Geofence' : 'Normal' }}
                      </span>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <!-- FUNCTIONAL NOTIFICATION BUTTON -->
            <div 
              :class="['notification-badge', { 'has-alert': emergencyAlerts.length > 0 }]" 
              @click="toggleNotifications"
              title="Toggle Safety Emergency Center"
            >
              <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                <path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"/><path d="M13.73 21a2 2 0 0 1-3.46 0"/>
              </svg>
              <span v-if="emergencyAlerts.length > 0" class="badge-count">{{ emergencyAlerts.length }}</span>
            </div>
          </div>
        </header>

        <!-- DYNAMIC MAIN TAB CONTENTS -->
        <div class="content-scrollable">
          
          <!-- Active Search Query Banner Indicator -->
          <div v-if="searchQuery" class="search-active-banner">
            <span>Filtering records matching <strong>"{{ searchQuery }}"</strong></span>
            <button @click="searchQuery = ''">Clear Search Filter</button>
          </div>

          <!-- 1. PUBLIC INFORMATION HUB -->
          <div v-if="activeTab === 'hub'" class="hub-container">
            <div class="filter-pills">
              <span>Filter by:</span>
              <button :class="{ active: hubFilter === 'all' }" @click="hubFilter = 'all'">All</button>
              <button :class="{ active: hubFilter === 'events' }" @click="hubFilter = 'events'">Events</button>
              <button :class="{ active: hubFilter === 'facilities' }" @click="hubFilter = 'facilities'">Facilities</button>
              <button :class="{ active: hubFilter === 'history' }" @click="hubFilter = 'history'">History</button>
            </div>

            <div class="card-list">
              <div class="feature-card" v-if="(hubFilter === 'all' || hubFilter === 'history') && matchesSearch('People\'s Park San Carlos City history eco-recreational coastal boulevard')">
                <div class="card-badge bg-blue">History & Info</div>
                <h3>People's Park San Carlos City</h3>
                <p>A flagship eco-recreational destination located along the coastal boulevard of San Carlos City. Built as a community sanctuary offering scenic oceanfront views, multi-purpose pavilions, and state-of-the-art aquatic recreation facility.</p>
                <div class="card-footer">
                  <span class="meta-tag">San Carlos City, Negros Occidental</span>
                </div>
              </div>

              <div class="feature-card" v-if="(hubFilter === 'all' || hubFilter === 'facilities') && matchesSearch('Facilities Park Zones Swimming Pool Pavilion')">
                <div class="card-badge bg-purple">Facilities Offered</div>
                <h3>Park Zones & Capacities</h3>
                <p><strong>1. Around the Park:</strong> Open-air trails, seaside promenade, playgrounds.<br>
                <strong>2. Swimming Pool:</strong> Olympic-sized public pool equipped with IoT safety tracking.<br>
                <strong>3. Pavilion:</strong> Covered event venue for gatherings and private functions.</p>
                <div class="card-footer">
                  <button class="action-btn" @click="activeTab = 'booking'">Book Facility</button>
                </div>
              </div>

              <div class="feature-card" v-if="(hubFilter === 'all' || hubFilter === 'events') && matchesSearch('Upcoming Community Fitness & Safety Orientation Zumba')">
                <div class="card-badge bg-rose">Events & Announcements</div>
                <h3>Upcoming Community Fitness & Safety Orientation</h3>
                <p>Join us this weekend for an open-air Zumba session around the park grounds followed by a public safety demonstration on our new IoT wristband monitoring system.</p>
                <div class="card-footer">
                  <span class="meta-tag">Schedule: Sat, 6:00 AM - 9:00 AM</span>
                </div>
              </div>
            </div>
          </div>

          <!-- 2. RESERVATION & BOOKING PIPELINE -->
          <div v-if="activeTab === 'booking'" class="booking-container">
            <div class="booking-card">
              <h3>Facility Reservation & Wristband Registration</h3>
              <p class="subtitle">Assign digital wristband tokens to your guests for entrance and safety tracking.</p>

              <form @submit.prevent="processBooking" class="booking-form">
                <div class="form-group">
                  <label>Select Facility</label>
                  <select v-model="bookingForm.facility" required>
                    <option value="Around the Park">Around the Park (General Admission)</option>
                    <option value="Swimming Pool">Swimming Pool</option>
                    <option value="Pavilion">Pavilion</option>
                  </select>
                </div>

                <div class="form-group">
                  <label>Number of Guests</label>
                  <input 
                    type="number" 
                    v-model.number="bookingForm.guestCount" 
                    min="1" 
                    max="10"
                    @change="updateGuestFields" 
                    required 
                  />
                </div>

                <div class="guest-input-list">
                  <label>Guest Name(s) & Auto-Assigned Wristband IDs</label>
                  <div v-for="(guest, idx) in bookingForm.guests" :key="idx" class="guest-row">
                    <input 
                      type="text" 
                      v-model="guest.name" 
                      :placeholder="`Guest #${idx+1} Full Name`" 
                      required 
                    />
                    <div class="wb-tag">
                      <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 2v20M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></svg>
                      {{ guest.wristbandId }}
                    </div>
                  </div>
                </div>

                <div class="form-group">
                  <label>Payment Method</label>
                  <div class="radio-options">
                    <label class="radio-label">
                      <input type="radio" value="E-Wallet" v-model="bookingForm.paymentMethod" />
                      E-Wallet (GCash/PayMaya)
                    </label>
                    <label class="radio-label">
                      <input type="radio" value="Cash" v-model="bookingForm.paymentMethod" />
                      Cash (Pay personally at front gate)
                    </label>
                  </div>
                </div>

                <button type="submit" class="submit-btn">Confirm Reservation & Issue Slip</button>
              </form>
            </div>

            <!-- Online Reservation Slip Modal -->
            <div v-if="latestReservation" class="slip-modal">
              <div class="slip-card">
                <div class="slip-header">
                  <h4>Online Reservation Slip</h4>
                  <span class="slip-status">Confirmed</span>
                </div>
                <div class="slip-body">
                  <p><strong>Reservation Code:</strong> {{ latestReservation.code }}</p>
                  <p><strong>Facility:</strong> {{ latestReservation.facility }}</p>
                  <p><strong>Payment Mode:</strong> {{ latestReservation.paymentMethod }}</p>
                  <div class="assigned-bands">
                    <p><strong>Assigned Wristband Tokens:</strong></p>
                    <ul>
                      <li v-for="g in latestReservation.guests" :key="g.wristbandId">
                        {{ g.name }} &rarr; <span class="badge-wb">{{ g.wristbandId }}</span>
                      </li>
                    </ul>
                  </div>
                  <p class="slip-note" v-if="latestReservation.paymentMethod === 'Cash'">
                    * Please present this slip and render payment at the park entrance to collect your physical wristbands.
                  </p>
                  <p class="slip-note" v-else>
                    * Receipt Generated! Claim physical wristbands at the receptionist counter upon arrival.
                  </p>
                </div>
                <button class="secondary-btn" @click="latestReservation = null">Close Slip</button>
              </div>
            </div>
          </div>

          <!-- 3. REAL-TIME LOCATION MONITORING -->
          <div v-if="activeTab === 'tracking'" class="tracking-container">
            <div class="map-card">
              <div class="map-header">
                <h3>Live Group Location Monitor</h3>
                <span class="active-tag">● Active Session: {{ currentGroupCode }}</span>
              </div>
              
              <div class="park-map">
                <div class="zone zone-park">Around the Park</div>
                <div class="zone zone-pool">Swimming Pool Zone</div>
                <div class="zone zone-pavilion">Pavilion</div>

                <div 
                  v-for="visitor in filteredVisitors" 
                  :key="visitor.wristbandId" 
                  class="map-pin"
                  :style="{ top: visitor.mapY + '%', left: visitor.mapX + '%' }"
                >
                  <div class="pin-bubble" :class="{ 'pin-alert': visitor.inDistress, 'pin-highlight': searchQuery && matchesVisitor(visitor) }">
                    <span class="pin-id">{{ visitor.wristbandId }}</span>
                    <span class="pin-status">{{ visitor.name }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- 4. RECEPTIONIST DASHBOARD -->
          <div v-if="activeTab === 'receptionist'" class="receptionist-container">
            <div class="dashboard-grid">
              <div class="stat-card">
                <h4>Active Park Visitors</h4>
                <div class="stat-value">{{ activeVisitors.length }}</div>
              </div>
              <div class="stat-card">
                <h4>Flagged / Invalid Tickets</h4>
                <div class="stat-value text-danger">{{ flaggedVisitors.length }}</div>
              </div>
            </div>

            <div class="table-card">
              <h3>Perimeter Integrity & Visitor Log</h3>
              <p class="subtitle">Monitors real-time geofence violations (leaving perimeter without checkout).</p>
              
              <table class="data-table">
                <thead>
                  <tr>
                    <th>Wristband ID</th>
                    <th>Guest Name</th>
                    <th>Zone</th>
                    <th>Geofence Status</th>
                    <th>Actions</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-if="filteredVisitors.length === 0">
                    <td colspan="5" class="text-center py-4 text-muted">No visitors found matching "{{ searchQuery }}"</td>
                  </tr>
                  <tr v-for="visitor in filteredVisitors" :key="visitor.wristbandId">
                    <td><strong class="badge-wb">{{ visitor.wristbandId }}</strong></td>
                    <td>{{ visitor.name }}</td>
                    <td>{{ visitor.zone }}</td>
                    <td>
                      <span :class="['status-pill', visitor.outOfBounds ? 'pill-invalid' : 'pill-valid']">
                        {{ visitor.outOfBounds ? 'INVALID (Outside Bounds)' : 'Normal (Inside Park)' }}
                      </span>
                    </td>
                    <td>
                      <button 
                        class="small-btn" 
                        @click="triggerCheckout(visitor.wristbandId)"
                      >
                        Checkout / Return Band
                      </button>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>

          <!-- 5. LIFEGUARD SAFETY & EMERGENCY DASHBOARD -->
          <div v-if="activeTab === 'lifeguard'" class="lifeguard-container">
            <div class="alert-banner" v-if="emergencyAlerts.length > 0">
              <div class="pulse-icon">🚨</div>
              <div>
                <h4>EMERGENCY ALERT DETECTED!</h4>
                <p>{{ emergencyAlerts.length }} guest(s) requiring immediate dispatch in the Swimming Pool area.</p>
              </div>
            </div>

            <div class="lifeguard-grid">
              <div class="card-list">
                <h3>Live Pool Biometric Telemetry</h3>
                <div 
                  v-for="swimmer in filteredPoolSwimmers" 
                  :key="swimmer.wristbandId" 
                  :class="['telemetry-card', { 'telemetry-danger': swimmer.inDistress }]"
                >
                  <div class="telemetry-head">
                    <span class="badge-wb">{{ swimmer.wristbandId }}</span>
                    <span class="zone-badge">{{ swimmer.zone }}</span>
                  </div>
                  <div class="telemetry-body">
                    <div class="metric">
                      <span>Guest Name:</span>
                      <strong>{{ swimmer.name }}</strong>
                    </div>
                    <div class="metric">
                      <span>Blood Oxygen (SpO₂):</span>
                      <strong :class="{ 'text-danger': swimmer.spo2 < 90 }">{{ swimmer.spo2 }}%</strong>
                    </div>
                    <div class="metric">
                      <span>Panic Button:</span>
                      <strong :class="{ 'text-danger': swimmer.panicPressed }">
                        {{ swimmer.panicPressed ? 'PRESSED (SOS)' : 'Normal' }}
                      </strong>
                    </div>
                  </div>
                  <div class="telemetry-actions" v-if="swimmer.inDistress">
                    <button class="sos-btn" @click="resolveEmergency(swimmer.wristbandId)">
                      Acknowledge & Resolve Emergency
                    </button>
                  </div>
                </div>
              </div>

              <!-- Simulation Controls -->
              <div class="sim-card">
                <h3>Safety Simulation Tools</h3>
                <p class="subtitle">Simulate real-time IoT wristband sensor events for testing:</p>
                
                <div class="sim-buttons">
                  <button class="sim-btn bg-danger" @click="simulateSpO2Drop">
                    Simulate SpO₂ Drop (&lt;90%)
                  </button>
                  <button class="sim-btn bg-warning" @click="simulatePanicButton">
                    Simulate Physical SOS Button Press
                  </button>
                  <button class="sim-btn bg-purple" @click="simulateGeofenceBreach">
                    Simulate Geofence Boundary Breach
                  </button>
                </div>
              </div>
            </div>
          </div>

        </div>
      </main>

      <!-- RIGHT PANEL (Calendar & Online Users Widget) -->
      <aside class="right-panel">
        <div class="user-profile">
          <div class="profile-info">
            <span class="user-name">Staff Admin</span>
            <span class="user-id">ID: PC-SCC-1094</span>
          </div>
          <div class="avatar">
            <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/>
            </svg>
          </div>
        </div>

        <!-- Calendar Widget -->
        <div class="calendar-widget">
          <div class="calendar-header">
            <h4>Sep 2026</h4>
            <div class="cal-nav">
              <span>&lt;</span>
              <span>&gt;</span>
            </div>
          </div>
          <div class="calendar-days-header">
            <span>M</span><span>T</span><span>W</span><span>T</span><span>F</span><span>S</span><span>S</span>
          </div>
          <div class="calendar-grid">
            <span class="dim">31</span><span>1</span><span>2</span><span>3</span><span>4</span><span>5</span><span>6</span>
            <span>7</span><span>8</span><span>9</span><span>10</span><span>11</span><span>12</span><span>13</span>
            <span>14</span><span>15</span><span>16</span><span>17</span><span>18</span><span>19</span><span>20</span>
            <span>21</span><span>22</span><span class="active-day">23</span><span>24</span><span>25</span><span>26</span><span>27</span>
            <span>28</span><span>29</span><span>30</span><span class="dim">1</span>
          </div>
        </div>

        <!-- Online Visitors Widget -->
        <div class="online-users-widget">
          <div class="widget-head">
            <h4>Active Bands On-Site</h4>
            <span class="see-all">Total: {{ activeVisitors.length }}</span>
          </div>

          <div class="user-list">
            <div v-for="visitor in filteredVisitors.slice(0, 5)" :key="visitor.wristbandId" class="user-item">
              <div class="user-avatar-circle">
                {{ visitor.wristbandId.slice(-2) }}
              </div>
              <div class="user-meta">
                <span class="u-name">{{ visitor.name }}</span>
                <span class="u-id">{{ visitor.wristbandId }} • {{ visitor.zone }}</span>
              </div>
              <span :class="['online-dot', { 'dot-alert': visitor.inDistress || visitor.outOfBounds }]"></span>
            </div>
          </div>
        </div>
      </aside>
    </div>

    <!-- NOTIFICATION SLIDE-OUT PANEL (LIFEGUARD EMERGENCY CENTER) -->
    <div v-if="showNotifications" class="notification-overlay" @click.self="showNotifications = false">
      <div class="notification-drawer">
        <div class="drawer-header">
          <div>
            <h3>Lifeguard Safety Alerts</h3>
            <span class="drawer-sub">Real-Time Distress & Emergency Queue</span>
          </div>
          <button class="close-drawer" @click="showNotifications = false">&times;</button>
        </div>

        <div class="drawer-body">
          <div v-if="emergencyAlerts.length === 0" class="empty-alerts">
            <div class="empty-icon">🛡️</div>
            <p>No active emergency alerts.</p>
            <span>All pool swimmers and visitors are currently within safe biometric thresholds.</span>
          </div>

          <div v-else class="alert-list">
            <div 
              v-for="alert in emergencyAlerts" 
              :key="alert.wristbandId" 
              class="alert-card"
            >
              <div class="alert-card-head">
                <span class="badge-wb-alert">{{ alert.wristbandId }}</span>
                <span class="alert-time">NOW</span>
              </div>
              <div class="alert-card-body">
                <h4>Distress Reported: {{ alert.name }}</h4>
                <p><strong>Location:</strong> {{ alert.zone }}</p>
                <p v-if="alert.spo2 < 90" class="alert-detail">
                  ⚠️ <strong>Critical SpO₂ Level:</strong> {{ alert.spo2 }}% (Potential Drowning)
                </p>
                <p v-if="alert.panicPressed" class="alert-detail">
                  🔴 <strong>Manual SOS Pressed:</strong> Physical Panic Button Triggered
                </p>
              </div>
              <div class="alert-card-actions">
                <button class="dispatch-btn" @click="goToLifeguardAndResolve(alert.wristbandId)">
                  Locate & Resolve Incident
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, reactive } from 'vue';

// Navigation & Search State
const activeTab = ref('hub');
const hubFilter = ref('all');
const searchQuery = ref('');
const isSearchFocused = ref(false);
const showNotifications = ref(false);

const navItems = [
  { 
    id: 'hub', 
    label: 'Information Hub', 
    icon: `<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/><rect x="14" y="14" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/></svg>` 
  },
  { 
    id: 'booking', 
    label: 'Reservation / Booking', 
    icon: `<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M4 19.5A2.5 2.5 0 0 1 6.5 17H20"/><path d="M6.5 2H20v20H6.5A2.5 2.5 0 0 1 4 19.5v-15A2.5 2.5 0 0 1 6.5 2z"/></svg>` 
  },
  { 
    id: 'tracking', 
    label: 'Location Monitoring', 
    icon: `<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"/><circle cx="12" cy="10" r="3"/></svg>` 
  },
];

const secondaryNavItems = [
  { 
    id: 'receptionist', 
    label: 'Receptionist Panel', 
    icon: `<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M17 21v-2a4 4 0 0 0-4-4H5a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M23 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></svg>` 
  },
  { 
    id: 'lifeguard', 
    label: 'Lifeguard Safety Portal', 
    icon: `<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 22s8-4 8-10V5l-8-3-8 3v7c0 6 8 10 8 10z"/></svg>` 
  },
];

const activeTabTitle = computed(() => {
  const all = [...navItems, ...secondaryNavItems];
  const current = all.find(item => item.id === activeTab.value);
  return current ? current.label : 'Dashboard';
});

// Booking Form State
const currentGroupCode = ref('GRP-8842');
const latestReservation = ref(null);
const bookingForm = reactive({
  facility: 'Swimming Pool',
  guestCount: 2,
  paymentMethod: 'E-Wallet',
  guests: [
    { name: 'Juan Dela Cruz', wristbandId: 'WB-101' },
    { name: 'Maria Dela Cruz', wristbandId: 'WB-102' }
  ]
});

// Active Visitors & Safety Tracking State
const activeVisitors = ref([
  { wristbandId: 'WB-101', name: 'Juan Dela Cruz', zone: 'Swimming Pool', spo2: 97, panicPressed: false, inDistress: false, outOfBounds: false, mapX: 45, mapY: 55 },
  { wristbandId: 'WB-102', name: 'Maria Dela Cruz', zone: 'Swimming Pool', spo2: 98, panicPressed: false, inDistress: false, outOfBounds: false, mapX: 52, mapY: 60 },
  { wristbandId: 'WB-103', name: 'Mark Anthony', zone: 'Around the Park', spo2: 99, panicPressed: false, inDistress: false, outOfBounds: false, mapX: 20, mapY: 30 },
  { wristbandId: 'WB-104', name: 'Sarah Lawson', zone: 'Pavilion', spo2: 96, panicPressed: false, inDistress: false, outOfBounds: false, mapX: 75, mapY: 25 },
]);

// SEARCH LOGIC & COMPUTED FILTERS
const matchesVisitor = (visitor) => {
  if (!searchQuery.value) return true;
  const q = searchQuery.value.toLowerCase().trim();
  return (
    visitor.name.toLowerCase().includes(q) ||
    visitor.wristbandId.toLowerCase().includes(q) ||
    visitor.zone.toLowerCase().includes(q)
  );
};

const matchesSearch = (text) => {
  if (!searchQuery.value) return true;
  return text.toLowerCase().includes(searchQuery.value.toLowerCase().trim());
};

const searchResults = computed(() => {
  if (!searchQuery.value.trim()) return [];
  return activeVisitors.value.filter(matchesVisitor);
});

const filteredVisitors = computed(() => {
  return activeVisitors.value.filter(matchesVisitor);
});

const filteredPoolSwimmers = computed(() => {
  return activeVisitors.value.filter(v => v.zone === 'Swimming Pool' && matchesVisitor(v));
});

const emergencyAlerts = computed(() => {
  return activeVisitors.value.filter(v => v.inDistress);
});

const flaggedVisitors = computed(() => {
  return activeVisitors.value.filter(v => v.outOfBounds);
});

const handleSearchBlur = () => {
  setTimeout(() => {
    isSearchFocused.value = false;
  }, 200);
};

const selectSearchResult = (res) => {
  if (res.zone === 'Swimming Pool' && res.inDistress) {
    activeTab.value = 'lifeguard';
  } else {
    activeTab.value = 'tracking';
  }
  isSearchFocused.value = false;
};

// Booking Methods
const updateGuestFields = () => {
  const count = bookingForm.guestCount;
  const currentGuests = [...bookingForm.guests];
  if (count > currentGuests.length) {
    for (let i = currentGuests.length; i < count; i++) {
      currentGuests.push({
        name: '',
        wristbandId: `WB-${101 + i}`
      });
    }
  } else {
    currentGuests.splice(count);
  }
  bookingForm.guests = currentGuests;
};

const processBooking = () => {
  latestReservation.value = {
    code: 'RES-' + Math.floor(100000 + Math.random() * 900000),
    facility: bookingForm.facility,
    paymentMethod: bookingForm.paymentMethod,
    guests: JSON.parse(JSON.stringify(bookingForm.guests))
  };

  bookingForm.guests.forEach((g, index) => {
    activeVisitors.value.push({
      wristbandId: g.wristbandId,
      name: g.name,
      zone: bookingForm.facility,
      spo2: 98,
      panicPressed: false,
      inDistress: false,
      outOfBounds: false,
      mapX: 30 + (index * 15),
      mapY: 40 + (index * 10)
    });
  });
};

// Notification & Simulation Methods
const toggleNotifications = () => {
  showNotifications.value = !showNotifications.value;
};

const goToLifeguardAndResolve = (id) => {
  activeTab.value = 'lifeguard';
  resolveEmergency(id);
  showNotifications.value = false;
};

const simulateSpO2Drop = () => {
  const swimmer = activeVisitors.value.find(v => v.zone === 'Swimming Pool');
  if (swimmer) {
    swimmer.spo2 = 84;
    swimmer.inDistress = true;
    showNotifications.value = true;
  }
};

const simulatePanicButton = () => {
  const swimmer = activeVisitors.value.find(v => v.zone === 'Swimming Pool');
  if (swimmer) {
    swimmer.panicPressed = true;
    swimmer.inDistress = true;
    showNotifications.value = true;
  }
};

const simulateGeofenceBreach = () => {
  const visitor = activeVisitors.value[0];
  if (visitor) {
    visitor.outOfBounds = true;
    visitor.mapX = 95;
    visitor.mapY = 95;
    activeTab.value = 'receptionist';
  }
};

const resolveEmergency = (id) => {
  const v = activeVisitors.value.find(item => item.wristbandId === id);
  if (v) {
    v.spo2 = 98;
    v.panicPressed = false;
    v.inDistress = false;
  }
};

const triggerCheckout = (id) => {
  activeVisitors.value = activeVisitors.value.filter(item => item.wristbandId !== id);
};
</script>

<style scoped>
/* BASE STYLES */
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
}

.app-viewport {
  width: 100vw;
  height: 100vh;
  background-color: #eef0f8;
  display: flex;
  justify-content: center;
  align-items: center;
  overflow: hidden;
  position: relative;
}

.app-container {
  width: 100vw;
  height: 100vh;
  background: #ffffff;
  display: grid;
  grid-template-columns: 240px 1fr 300px;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.08);
  overflow: hidden;
}

/* SIDEBAR */
.sidebar {
  background: #4d44b5;
  color: #ffffff;
  padding: 28px 20px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.brand {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 32px;
}

.brand-icon {
  background: rgba(255, 255, 255, 0.2);
  padding: 8px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.brand-text h2 {
  font-size: 16px;
  font-weight: 700;
  line-height: 1.2;
}

.brand-text span {
  font-size: 11px;
  opacity: 0.75;
}

.nav-menu, .sidebar-footer {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.nav-btn {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 12px 16px;
  background: transparent;
  border: none;
  color: rgba(255, 255, 255, 0.7);
  font-size: 14px;
  font-weight: 500;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
  text-align: left;
}

.nav-btn:hover {
  background: rgba(255, 255, 255, 0.1);
  color: #ffffff;
}

.nav-btn.active {
  background: #ffffff;
  color: #4d44b5;
  font-weight: 600;
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.nav-icon {
  display: flex;
  align-items: center;
}

/* MAIN CONTENT CANVAS */
.main-content {
  background-color: #f7f8fc;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  border-right: 1px solid #eef0f6;
}

.top-header {
  padding: 24px 32px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #f7f8fc;
}

.header-title h1 {
  font-size: 24px;
  font-weight: 700;
  color: #303972;
}

.header-title p {
  font-size: 13px;
  color: #a0a5ba;
  margin-top: 2px;
}

.header-actions {
  display: flex;
  align-items: center;
  gap: 16px;
}

/* INTERACTIVE SEARCH BAR & DROPDOWN */
.search-wrapper {
  position: relative;
}

.search-bar {
  display: flex;
  align-items: center;
  gap: 10px;
  background: #ffffff;
  padding: 10px 16px;
  border-radius: 30px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.03);
  color: #a0a5ba;
  border: 1px solid transparent;
  transition: all 0.2s ease;
}

.search-bar.is-focused {
  border-color: #4d44b5;
  box-shadow: 0 4px 14px rgba(77, 68, 181, 0.15);
}

.search-bar input {
  border: none;
  outline: none;
  font-size: 13px;
  width: 240px;
}

.clear-search {
  background: transparent;
  border: none;
  color: #a0a5ba;
  font-size: 16px;
  cursor: pointer;
}

.search-results-dropdown {
  position: absolute;
  top: 48px;
  left: 0;
  width: 320px;
  background: #ffffff;
  border-radius: 16px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.12);
  z-index: 100;
  padding: 12px;
  border: 1px solid #f1f5f9;
}

.no-results {
  padding: 16px;
  font-size: 13px;
  color: #64748b;
  text-align: center;
}

.results-group-title {
  font-size: 11px;
  font-weight: 700;
  color: #94a3b8;
  text-transform: uppercase;
  margin-bottom: 8px;
  padding-left: 8px;
}

.search-result-item {
  padding: 10px 12px;
  border-radius: 10px;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: background 0.2s ease;
}

.search-result-item:hover {
  background: #f8fafc;
}

.res-info {
  display: flex;
  align-items: center;
  gap: 8px;
}

.res-name {
  font-size: 13px;
  color: #334155;
}

.res-meta {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 2px;
}

.res-zone {
  font-size: 11px;
  color: #94a3b8;
}

.status-pill-sm {
  font-size: 9px;
  padding: 2px 6px;
  border-radius: 6px;
  font-weight: 700;
}

.pill-ok { background: #dcfce7; color: #15803d; }
.pill-danger { background: #fee2e2; color: #b91c1c; }
.pill-warning { background: #fef3c7; color: #b45309; }

.search-active-banner {
  background: #e0e7ff;
  color: #3730a3;
  padding: 10px 16px;
  border-radius: 12px;
  margin-bottom: 16px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 13px;
}

.search-active-banner button {
  background: transparent;
  border: none;
  color: #4338ca;
  font-weight: 700;
  cursor: pointer;
  text-decoration: underline;
}

/* INTERACTIVE NOTIFICATION BADGE */
.notification-badge {
  background: #ffffff;
  padding: 10px;
  border-radius: 50%;
  position: relative;
  cursor: pointer;
  color: #a0a5ba;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.03);
  transition: transform 0.2s ease, background 0.2s ease;
}

.notification-badge:hover {
  transform: scale(1.08);
  color: #4d44b5;
}

.notification-badge.has-alert {
  color: #ef4444;
  animation: ring 1.5s infinite;
}

@keyframes ring {
  0% { transform: rotate(0); }
  10% { transform: rotate(15deg); }
  20% { transform: rotate(-10deg); }
  30% { transform: rotate(5deg); }
  40% { transform: rotate(0); }
  100% { transform: rotate(0); }
}

.badge-count {
  position: absolute;
  top: 2px;
  right: 2px;
  background: #ef4444;
  color: #ffffff;
  font-size: 10px;
  font-weight: 700;
  padding: 2px 6px;
  border-radius: 10px;
}

.content-scrollable {
  padding: 0 32px 32px 32px;
  overflow-y: auto;
  flex: 1;
}

/* FILTER PILLS & CARDS */
.filter-pills {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 20px;
  font-size: 13px;
  color: #a0a5ba;
}

.filter-pills button {
  background: #ffffff;
  border: 1px solid #e0e4f0;
  padding: 6px 16px;
  border-radius: 20px;
  font-size: 13px;
  color: #303972;
  cursor: pointer;
}

.filter-pills button.active {
  background: #4d44b5;
  color: #ffffff;
  border-color: #4d44b5;
}

.card-list {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.feature-card {
  background: #ffffff;
  padding: 24px;
  border-radius: 20px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.03);
  position: relative;
}

.card-badge {
  display: inline-block;
  padding: 4px 12px;
  border-radius: 12px;
  font-size: 11px;
  font-weight: 600;
  margin-bottom: 12px;
}

.bg-blue { background: #e8f0fe; color: #1a73e8; }
.bg-purple { background: #f3e8ff; color: #7e22ce; }
.bg-rose { background: #ffe4e6; color: #e11d48; }

.feature-card h3 {
  font-size: 18px;
  color: #303972;
  margin-bottom: 8px;
}

.feature-card p {
  font-size: 14px;
  color: #667085;
  line-height: 1.6;
}

.card-footer {
  margin-top: 16px;
  display: flex;
  justify-content: flex-end;
}

.meta-tag {
  font-size: 12px;
  color: #a0a5ba;
}

/* BOOKING & SLIP */
.booking-card {
  background: #ffffff;
  padding: 28px;
  border-radius: 20px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.03);
}

.subtitle {
  font-size: 13px;
  color: #a0a5ba;
  margin-bottom: 20px;
}

.booking-form {
  display: flex;
  flex-direction: column;
  gap: 18px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.form-group label {
  font-size: 13px;
  font-weight: 600;
  color: #303972;
}

.form-group input, .form-group select {
  padding: 12px 16px;
  border: 1px solid #e2e8f0;
  border-radius: 12px;
  outline: none;
  font-size: 14px;
}

.guest-input-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.guest-row {
  display: flex;
  align-items: center;
  gap: 12px;
}

.guest-row input {
  flex: 1;
  padding: 10px 14px;
  border: 1px solid #e2e8f0;
  border-radius: 10px;
}

.wb-tag {
  background: #f1f5f9;
  padding: 8px 12px;
  border-radius: 10px;
  font-size: 12px;
  font-weight: 600;
  color: #4d44b5;
  display: flex;
  align-items: center;
  gap: 6px;
}

.radio-options {
  display: flex;
  gap: 20px;
}

.radio-label {
  font-size: 13px;
  color: #475569;
  display: flex;
  align-items: center;
  gap: 6px;
  cursor: pointer;
}

.submit-btn, .action-btn {
  background: #4d44b5;
  color: #ffffff;
  padding: 12px 24px;
  border: none;
  border-radius: 12px;
  font-weight: 600;
  cursor: pointer;
  align-self: flex-start;
}

.slip-modal {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.4);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 100;
}

.slip-card {
  background: #ffffff;
  padding: 28px;
  border-radius: 20px;
  width: 420px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
}

.slip-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  border-bottom: 1px solid #eee;
  padding-bottom: 12px;
}

.slip-status {
  background: #dcfce7;
  color: #15803d;
  font-size: 11px;
  padding: 4px 8px;
  border-radius: 8px;
  font-weight: 700;
}

.slip-body {
  display: flex;
  flex-direction: column;
  gap: 10px;
  font-size: 14px;
  color: #334155;
}

.slip-note {
  font-size: 12px;
  color: #fb7d5b;
  margin-top: 8px;
}

.secondary-btn {
  margin-top: 20px;
  width: 100%;
  padding: 10px;
  background: #f1f5f9;
  border: none;
  border-radius: 10px;
  font-weight: 600;
  color: #475569;
  cursor: pointer;
}

/* REAL TIME PARK MAP */
.map-card {
  background: #ffffff;
  padding: 24px;
  border-radius: 20px;
}

.map-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 16px;
}

.active-tag {
  font-size: 12px;
  color: #16a34a;
  font-weight: 600;
}

.park-map {
  width: 100%;
  height: 380px;
  background: #e2e8f0;
  border-radius: 16px;
  position: relative;
  overflow: hidden;
  border: 2px dashed #cbd5e1;
}

.zone {
  position: absolute;
  padding: 8px 14px;
  background: rgba(255,255,255,0.85);
  border-radius: 8px;
  font-size: 11px;
  font-weight: 700;
  color: #475569;
}

.zone-park { top: 20px; left: 20px; }
.zone-pool { top: 140px; left: 160px; background: rgba(224, 242, 254, 0.9); color: #0369a1; }
.zone-pavilion { bottom: 30px; right: 30px; }

.map-pin {
  position: absolute;
  transform: translate(-50%, -50%);
  transition: all 0.5s ease;
}

.pin-bubble {
  background: #4d44b5;
  color: white;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
  box-shadow: 0 4px 10px rgba(0,0,0,0.15);
}

.pin-alert {
  background: #ef4444;
  animation: pulse 1s infinite alternate;
}

.pin-highlight {
  background: #f59e0b;
  transform: scale(1.2);
}

@keyframes pulse {
  from { transform: scale(1); }
  to { transform: scale(1.15); }
}

/* TABLES & PANELS */
.receptionist-container, .lifeguard-container {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.dashboard-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.stat-card {
  background: #ffffff;
  padding: 20px;
  border-radius: 16px;
}

.stat-card h4 {
  font-size: 13px;
  color: #a0a5ba;
}

.stat-value {
  font-size: 28px;
  font-weight: 700;
  color: #303972;
  margin-top: 8px;
}

.table-card {
  background: #ffffff;
  padding: 24px;
  border-radius: 20px;
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 16px;
}

.data-table th, .data-table td {
  text-align: left;
  padding: 12px 14px;
  font-size: 13px;
  border-bottom: 1px solid #f1f5f9;
}

.status-pill {
  padding: 4px 10px;
  border-radius: 12px;
  font-size: 11px;
  font-weight: 600;
}

.pill-valid { background: #dcfce7; color: #166534; }
.pill-invalid { background: #fee2e2; color: #991b1b; }

.small-btn {
  padding: 6px 12px;
  background: #f1f5f9;
  border: none;
  border-radius: 8px;
  font-size: 11px;
  cursor: pointer;
}

/* LIFEGUARD PORTAL */
.alert-banner {
  background: #fee2e2;
  border: 1px solid #fca5a5;
  padding: 16px;
  border-radius: 16px;
  display: flex;
  align-items: center;
  gap: 16px;
  color: #991b1b;
}

.lifeguard-grid {
  display: grid;
  grid-template-columns: 1fr 280px;
  gap: 20px;
}

.telemetry-card {
  background: #ffffff;
  border-radius: 16px;
  padding: 16px;
  border: 1px solid #e2e8f0;
}

.telemetry-danger {
  border-color: #ef4444;
  background: #fef2f2;
}

.telemetry-head {
  display: flex;
  justify-content: space-between;
  margin-bottom: 12px;
}

.zone-badge {
  font-size: 11px;
  color: #64748b;
}

.telemetry-body {
  display: flex;
  gap: 20px;
  font-size: 13px;
}

.sos-btn {
  margin-top: 12px;
  width: 100%;
  background: #dc2626;
  color: white;
  border: none;
  padding: 8px;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
}

.sim-card {
  background: #ffffff;
  padding: 20px;
  border-radius: 16px;
  height: fit-content;
}

.sim-buttons {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-top: 16px;
}

.sim-btn {
  padding: 10px;
  border: none;
  border-radius: 8px;
  color: white;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
}

.bg-danger { background: #ef4444; }
.bg-warning { background: #f59e0b; }

/* RIGHT PANEL */
.right-panel {
  background: #ffffff;
  padding: 28px 20px;
  display: flex;
  flex-direction: column;
  gap: 28px;
}

.user-profile {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  gap: 12px;
}

.profile-info {
  text-align: right;
}

.user-name {
  display: block;
  font-size: 14px;
  font-weight: 700;
  color: #303972;
}

.user-id {
  font-size: 11px;
  color: #a0a5ba;
}

.avatar {
  background: #c1bbeb;
  padding: 8px;
  border-radius: 50%;
  color: #4d44b5;
}

.calendar-widget {
  background: #ffffff;
}

.calendar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}

.calendar-header h4 {
  color: #303972;
  font-size: 16px;
}

.cal-nav span {
  cursor: pointer;
  padding: 0 4px;
  color: #a0a5ba;
}

.calendar-days-header, .calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  text-align: center;
  font-size: 12px;
  gap: 8px 0;
}

.calendar-days-header {
  color: #a0a5ba;
  font-weight: 600;
  margin-bottom: 8px;
}

.calendar-grid span {
  padding: 6px 0;
  color: #303972;
  font-size: 12px;
}

.calendar-grid .dim {
  color: #d1d5db;
}

.calendar-grid .active-day {
  background: #4d44b5;
  color: #ffffff;
  border-radius: 50%;
  font-weight: 700;
}

.online-users-widget {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.widget-head {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.widget-head h4 {
  font-size: 15px;
  color: #303972;
}

.see-all {
  font-size: 12px;
  color: #a0a5ba;
}

.user-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.user-item {
  display: flex;
  align-items: center;
  gap: 12px;
}

.user-avatar-circle {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: #e8f0fe;
  color: #1a73e8;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  font-weight: 700;
}

.user-meta {
  flex: 1;
  display: flex;
  flex-direction: column;
}

.u-name {
  font-size: 13px;
  font-weight: 600;
  color: #303972;
}

.u-id {
  font-size: 11px;
  color: #a0a5ba;
}

.online-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #22c55e;
}

.dot-alert {
  background: #ef4444;
}

.text-danger { color: #dc2626; }
.text-muted { color: #a0a5ba; }
.text-center { text-align: center; }
.badge-wb { background: #e0e7ff; color: #3730a3; padding: 2px 6px; border-radius: 6px; font-size: 11px; }

/* NOTIFICATION DRAWER STYLES */
.notification-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.3);
  z-index: 200;
  display: flex;
  justify-content: flex-end;
}

.notification-drawer {
  width: 380px;
  height: 100%;
  background: #ffffff;
  box-shadow: -10px 0 30px rgba(0,0,0,0.1);
  display: flex;
  flex-direction: column;
  animation: slideIn 0.25s ease-out;
}

@keyframes slideIn {
  from { transform: translateX(100%); }
  to { transform: translateX(0); }
}

.drawer-header {
  padding: 24px;
  border-bottom: 1px solid #f1f5f9;
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
}

.drawer-header h3 {
  font-size: 18px;
  color: #303972;
}

.drawer-sub {
  font-size: 12px;
  color: #a0a5ba;
}

.close-drawer {
  background: transparent;
  border: none;
  font-size: 24px;
  color: #94a3b8;
  cursor: pointer;
}

.drawer-body {
  padding: 20px;
  overflow-y: auto;
  flex: 1;
}

.empty-alerts {
  text-align: center;
  padding: 40px 20px;
  color: #64748b;
}

.empty-icon {
  font-size: 40px;
  margin-bottom: 12px;
}

.empty-alerts p {
  font-weight: 600;
  color: #303972;
}

.empty-alerts span {
  font-size: 12px;
  color: #94a3b8;
}

.alert-list {
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.alert-card {
  background: #fef2f2;
  border: 1px solid #fca5a5;
  border-radius: 14px;
  padding: 16px;
}

.alert-card-head {
  display: flex;
  justify-content: space-between;
  margin-bottom: 8px;
}

.badge-wb-alert {
  background: #ef4444;
  color: white;
  padding: 2px 8px;
  border-radius: 6px;
  font-weight: 700;
  font-size: 11px;
}

.alert-time {
  font-size: 10px;
  font-weight: 700;
  color: #dc2626;
}

.alert-card-body h4 {
  font-size: 14px;
  color: #991b1b;
  margin-bottom: 6px;
}

.alert-card-body p {
  font-size: 12px;
  color: #7f1d1d;
  margin-bottom: 4px;
}

.dispatch-btn {
  width: 100%;
  margin-top: 10px;
  padding: 8px;
  background: #dc2626;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
}
</style>