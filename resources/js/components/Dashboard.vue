<template>
    <div class="ea-container">

        <!-- Header with Controls -->
        <div class="ea-header">
            <div class="ea-controls">
                <select v-model="dateRange" @change="onDateRangeChange" class="ea-select">
                    <option value="24hours">Last 24 Hours</option>
                    <option value="7days">Last 7 Days</option>
                    <option value="30days">Last 30 Days</option>
                    <option value="custom">Custom Range</option>
                </select>
                <div v-if="dateRange === 'custom'" class="ea-controls">
                    <input type="date" v-model="startDate" @change="fetchData" class="ea-input">
                    <span class="ea-text-lg">to</span>
                    <input type="date" v-model="endDate" @change="fetchData" class="ea-input">
                </div>
            </div>
            <div class="ea-controls">
                <button @click="fetchData" class="ea-btn ea-btn-primary">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"/>
                    </svg>
                    Refresh Data
                </button>
                <button @click="exportData" class="ea-btn ea-btn-success">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 10v6m0 0l-3-3m3 3l3-3m2 8H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>
                    </svg>
                    Export Data
                </button>
                <button @click="showSettings = !showSettings" class="ea-btn">
                    <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"/>
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/>
                    </svg>
                    Settings
                </button>
            </div>
        </div>

        <!-- Settings Panel -->
        <div v-if="showSettings" class="ea-card">
            <h3 class="ea-text-xl ea-mb-4">Analytics Settings</h3>
            <div class="ea-grid ea-grid-cols-2 ea-gap-8">
                <div class="ea-space-y-4">
                    <h4 class="ea-text-lg ea-font-semibold">Geolocation Stats</h4>
                    <div class="ea-space-y-2">
                        <div class="ea-flex ea-justify-between">
                            <span class="ea-text-secondary">Total Lookups:</span>
                            <span class="ea-font-medium">{{ geoStats.total_lookups.toLocaleString() }}</span>
                        </div>
                        <div class="ea-flex ea-justify-between">
                            <span class="ea-text-secondary">Success Rate:</span>
                            <span class="ea-font-medium">{{ geoSuccessRate }}</span>
                        </div>
                        <div class="ea-flex ea-justify-between">
                            <span class="ea-text-secondary">Unique IPs:</span>
                            <span class="ea-font-medium">{{ geoStats.unique_ips.length.toLocaleString() }}</span>
                        </div>
                        <div class="ea-flex ea-justify-between">
                            <span class="ea-text-secondary">Last Lookup:</span>
                            <span class="ea-font-medium">{{ geoLastLookup }}</span>
                        </div>
                    </div>
                    <button @click="clearGeoCache" :disabled="clearingCache" class="ea-btn ea-btn-primary ea-mt-4">
                        {{ clearingCache ? 'Clearing...' : 'Clear Geo Cache' }}
                    </button>
                </div>
                <div class="ea-space-y-4">
                    <h4 class="ea-text-lg ea-font-semibold">Current Configuration</h4>
                    <div class="ea-space-y-2">
                        <div class="ea-flex ea-justify-between">
                            <span class="ea-text-secondary">Cache Duration:</span>
                            <span class="ea-font-medium">{{ config.cacheDuration }} minutes</span>
                        </div>
                        <div class="ea-flex ea-justify-between">
                            <span class="ea-text-secondary">Rate Limit:</span>
                            <span class="ea-font-medium">{{ config.rateLimit }} requests/minute</span>
                        </div>
                        <div class="ea-flex ea-justify-between">
                            <span class="ea-text-secondary">Processing:</span>
                            <span class="ea-font-medium">Every {{ config.processingFrequency }} minutes</span>
                        </div>
                        <div class="ea-flex ea-justify-between">
                            <span class="ea-text-secondary">Dashboard Refresh:</span>
                            <span class="ea-font-medium">{{ config.refreshInterval }} seconds</span>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Quick Stats Overview -->
        <div class="ea-grid ea-grid-cols-4">
            <div class="ea-card">
                <h3 class="ea-font-semibold">Total Visits</h3>
                <p class="ea-text-lg">{{ overview.totalVisits.toLocaleString() }}</p>
                <p :class="['ea-text-secondary', 'text-sm', comparisonClass(overview.comparisons.total_visits, true)]">
                    {{ comparisonText(overview.comparisons.total_visits) }}
                </p>
            </div>
            <div class="ea-card">
                <h3 class="ea-font-semibold">Unique Visitors</h3>
                <p class="ea-text-lg">{{ overview.uniqueVisitors.toLocaleString() }}</p>
                <p :class="['ea-text-secondary', 'text-sm', comparisonClass(overview.comparisons.unique_visitors, true)]">
                    {{ comparisonText(overview.comparisons.unique_visitors) }}
                </p>
            </div>
            <div class="ea-card">
                <h3 class="ea-font-semibold">Engagement</h3>
                <p class="ea-text-lg">{{ formatDuration(overview.avgTimeOnSite) }}</p>
                <p class="ea-text-secondary">avg. time on site</p>
            </div>
            <div class="ea-card">
                <h3 class="ea-font-semibold">Bounce Rate</h3>
                <p class="ea-text-lg">{{ formatPercent(overview.bounceRate) }}</p>
                <p :class="['ea-text-secondary', 'text-sm', comparisonClass(overview.comparisons.bounce_rate, false)]">
                    {{ comparisonText(overview.comparisons.bounce_rate) }}
                </p>
            </div>
        </div>

        <!-- Visitor Engagement Metrics -->
        <div class="ea-grid ea-grid-cols-2">
            <div class="ea-card">
                <h3 class="ea-font-bold">Visit Frequency</h3>
                <div class="ea-grid ea-grid-cols-2">
                    <div>
                        <p class="ea-text-secondary">New Visitors</p>
                        <p class="ea-text-lg">{{ engagement.newVisitors.toLocaleString() }}</p>
                    </div>
                    <div>
                        <p class="ea-text-secondary">Returning Visitors</p>
                        <p class="ea-text-lg">{{ engagement.returningVisitors.toLocaleString() }}</p>
                    </div>
                    <div>
                        <p class="ea-text-secondary">Pages/Session</p>
                        <p class="ea-text-lg">{{ engagement.pagesPerSession.toFixed(1) }}</p>
                    </div>
                    <div>
                        <p class="ea-text-secondary">Avg. Session Duration</p>
                        <p class="ea-text-lg">{{ formatDuration(engagement.avgSessionDuration) }}</p>
                    </div>
                </div>
            </div>
            <div class="ea-card">
                <h3 class="ea-font-bold">Page Views Over Time</h3>
                <div class="ea-chart-wrapper">
                    <canvas ref="pageViewsChartEl"></canvas>
                </div>
            </div>
        </div>

        <!-- Geographic & Technical Insights -->
        <div class="ea-grid ea-grid-cols-1">
            <div class="ea-card">
                <h3 class="ea-font-bold">Top Countries</h3>
                <div class="ea-chart-wrapper">
                    <canvas ref="countryChartEl"></canvas>
                </div>
                <div>
                    <table class="ea-table">
                        <thead>
                            <tr>
                                <th>Country</th>
                                <th class="ea-text-right">Visits</th>
                                <th class="ea-text-right">% of Total</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="country in countryStats" :key="country.dimension_value">
                                <td>{{ country.dimension_value }}</td>
                                <td class="ea-text-right">{{ country.total.toLocaleString() }}</td>
                                <td class="ea-text-right">{{ countryPercent(country.total) }}</td>
                            </tr>
                        </tbody>
                    </table>
                </div>
            </div>
            <div class="ea-card">
                <h3 class="ea-font-bold">Device Types</h3>
                <div class="ea-chart-wrapper">
                    <canvas ref="deviceChartEl"></canvas>
                </div>
            </div>
            <div class="ea-card">
                <h3 class="ea-font-bold">Browser Usage</h3>
                <div class="ea-chart-wrapper">
                    <canvas ref="browserChartEl"></canvas>
                </div>
            </div>
        </div>

        <!-- Page Performance -->
        <div class="ea-card">
            <h3 class="ea-font-bold">Page Performance</h3>
            <div class="overflow-x-auto">
                <table class="ea-table">
                    <thead>
                        <tr>
                            <th>Page URL</th>
                            <th>Views</th>
                            <th>Unique Views</th>
                            <th>Avg. Time</th>
                            <th>Bounce Rate</th>
                            <th>Exit Rate</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-if="topPages.length === 0">
                            <td colspan="6" class="ea-text-center">Loading data...</td>
                        </tr>
                        <tr v-for="page in topPages" :key="page.page_url">
                            <td class="max-w-md truncate">{{ page.page_url }}</td>
                            <td>{{ page.views.toLocaleString() }}</td>
                            <td>{{ page.unique_views.toLocaleString() }}</td>
                            <td>{{ formatDuration(page.avg_time) }}</td>
                            <td>{{ formatPercent(page.bounce_rate) }}</td>
                            <td>{{ formatPercent(page.exit_rate) }}</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

        <!-- User Flow -->
        <div class="ea-card">
            <h3 class="ea-font-bold">User Flow</h3>
            <div class="ea-grid ea-grid-cols-3">
                <div>
                    <h4 class="ea-font-semibold">Top Entry Pages</h4>
                    <p class="ea-text-muted">Entry points</p>
                    <div v-for="page in userFlow.entry_pages" :key="page.page_url" class="flex justify-between items-center">
                        <span class="truncate">{{ page.page_url }}</span>
                        <span class="text-gray-500">{{ page.count.toLocaleString() }}</span>
                    </div>
                </div>
                <div>
                    <h4 class="ea-font-semibold">Most Engaged Pages</h4>
                    <p class="ea-text-muted">Highest engagement</p>
                    <div v-for="page in userFlow.engaged_pages" :key="page.url" class="flex justify-between items-center">
                        <span class="truncate">{{ page.url }}</span>
                        <span class="text-gray-500">{{ formatDuration(page.avg_time) }}</span>
                    </div>
                </div>
                <div>
                    <h4 class="ea-font-semibold">Top Exit Pages</h4>
                    <p class="ea-text-muted">Exit points</p>
                    <div v-for="page in userFlow.exit_pages" :key="page.url" class="flex justify-between items-center">
                        <span class="truncate">{{ page.url }}</span>
                        <span class="text-gray-500">{{ formatPercent(page.exit_rate) }}</span>
                    </div>
                </div>
            </div>
        </div>

    </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue'
import { Chart } from 'chart.js/auto'

const props = defineProps({
    config: {
        type: Object,
        required: true,
    },
})

// UI state
const dateRange  = ref('7days')
const startDate  = ref('')
const endDate    = ref('')
const showSettings  = ref(false)
const clearingCache = ref(false)

// Data state
const overview = reactive({
    totalVisits:    0,
    uniqueVisitors: 0,
    avgTimeOnSite:  0,
    bounceRate:     0,
    comparisons: { total_visits: 0, unique_visitors: 0, bounce_rate: 0 },
})

const engagement = reactive({
    newVisitors:        0,
    returningVisitors:  0,
    pagesPerSession:    0,
    avgSessionDuration: 0,
})

const geoStats = reactive({
    total_lookups:      0,
    successful_lookups: 0,
    unique_ips:         [],
    last_lookup:        null,
})

const countryStats = ref([])
const topPages     = ref([])
const userFlow     = reactive({ entry_pages: [], engaged_pages: [], exit_pages: [] })

// Chart canvas refs
const pageViewsChartEl = ref(null)
const deviceChartEl    = ref(null)
const countryChartEl   = ref(null)
const browserChartEl   = ref(null)

// Chart instances
let pageViewsChart = null
let deviceChart    = null
let countryChart   = null
let browserChart   = null

// Auto-refresh timer
let refreshTimer = null

// ── Computed ──────────────────────────────────────────────────────────────────

const geoSuccessRate = computed(() => {
    if (!geoStats.total_lookups) return '0%'
    return ((geoStats.successful_lookups / geoStats.total_lookups) * 100).toFixed(1) + '%'
})

const geoLastLookup = computed(() =>
    geoStats.last_lookup ? new Date(geoStats.last_lookup).toLocaleString() : 'Never'
)

// ── Helpers ───────────────────────────────────────────────────────────────────

function formatDuration(seconds) {
    if (!seconds) return '0:00'
    const m = Math.floor(seconds / 60)
    const s = Math.floor(seconds % 60)
    return `${m}:${s.toString().padStart(2, '0')}`
}

function formatPercent(value) {
    return `${(value * 100).toFixed(1)}%`
}

function countryPercent(total) {
    return `${((total / (overview.totalVisits || 1)) * 100).toFixed(1)}%`
}

function comparisonClass(value, positiveGood) {
    const isPositive = value >= 0
    return (isPositive && positiveGood) || (!isPositive && !positiveGood)
        ? 'text-green-600'
        : 'text-red-600'
}

function comparisonText(value) {
    return `${value >= 0 ? '+' : ''}${value}% vs previous period`
}

function csrfToken() {
    return document.querySelector('meta[name="csrf-token"]')?.getAttribute('content') ?? ''
}

// ── Charts ────────────────────────────────────────────────────────────────────

function initCharts() {
    pageViewsChart = new Chart(pageViewsChartEl.value, {
        type: 'line',
        data: {
            labels: [],
            datasets: [
                { label: 'Total Views',  data: [], borderColor: 'rgb(59, 130, 246)', tension: 0.1 },
                { label: 'Unique Views', data: [], borderColor: 'rgb(16, 185, 129)', tension: 0.1 },
            ],
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            interaction: { intersect: false, mode: 'index' },
        },
    })

    deviceChart = new Chart(deviceChartEl.value, {
        type: 'doughnut',
        data: {
            labels: [],
            datasets: [{ data: [], backgroundColor: ['rgb(59, 130, 246)', 'rgb(16, 185, 129)', 'rgb(251, 191, 36)'] }],
        },
        options: { responsive: true, maintainAspectRatio: false },
    })

    countryChart = new Chart(countryChartEl.value, {
        type: 'bar',
        data: {
            labels: [],
            datasets: [{ label: 'Visits', data: [], backgroundColor: 'rgb(59, 130, 246)' }],
        },
        options: { responsive: true, maintainAspectRatio: false, indexAxis: 'y' },
    })

    browserChart = new Chart(browserChartEl.value, {
        type: 'doughnut',
        data: {
            labels: [],
            datasets: [{
                data: [],
                backgroundColor: [
                    'rgb(59, 130, 246)', 'rgb(16, 185, 129)', 'rgb(251, 191, 36)',
                    'rgb(236, 72, 153)', 'rgb(124, 58, 237)',
                ],
            }],
        },
        options: { responsive: true, maintainAspectRatio: false },
    })
}

// ── Data fetching ─────────────────────────────────────────────────────────────

async function fetchData() {
    try {
        const params = new URLSearchParams({ range: dateRange.value })
        if (dateRange.value === 'custom' && startDate.value && endDate.value) {
            params.append('start_date', startDate.value)
            params.append('end_date', endDate.value)
        }

        const response = await fetch(`${props.config.routes.data}?${params}`)
        if (!response.ok) throw new Error('Failed to fetch analytics data')

        const data = await response.json()
        updateState(data)
        await fetchGeoStats()
    } catch (error) {
        console.error('Error fetching analytics data:', error)
    }
}

function updateState(data) {
    if (data.overview) {
        overview.totalVisits    = data.overview.total_visits
        overview.uniqueVisitors = data.overview.unique_visitors
        overview.avgTimeOnSite  = data.overview.avg_time_on_site
        overview.bounceRate     = data.overview.bounce_rate
        if (data.overview.comparisons) {
            Object.assign(overview.comparisons, data.overview.comparisons)
        }
    }

    if (data.engagement) {
        engagement.newVisitors        = data.engagement.new_visitors
        engagement.returningVisitors  = data.engagement.returning_visitors
        engagement.pagesPerSession    = data.engagement.pages_per_session
        engagement.avgSessionDuration = data.engagement.avg_session_duration
    }

    if (data.top_pages)    topPages.value = data.top_pages
    if (data.country_stats) countryStats.value = data.country_stats

    if (data.user_flow) {
        userFlow.entry_pages   = data.user_flow.entry_pages   ?? []
        userFlow.engaged_pages = data.user_flow.engaged_pages ?? []
        userFlow.exit_pages    = data.user_flow.exit_pages    ?? []
    }

    if (data.page_views) {
        pageViewsChart.data.labels                = data.page_views.map(i => i.date)
        pageViewsChart.data.datasets[0].data      = data.page_views.map(i => i.total_views)
        pageViewsChart.data.datasets[1].data      = data.page_views.map(i => i.unique_views)
        pageViewsChart.update()
    }
    if (data.device_stats) {
        deviceChart.data.labels              = data.device_stats.map(i => i.dimension_value)
        deviceChart.data.datasets[0].data    = data.device_stats.map(i => i.total)
        deviceChart.update()
    }
    if (data.country_stats) {
        countryChart.data.labels             = data.country_stats.map(i => i.dimension_value)
        countryChart.data.datasets[0].data   = data.country_stats.map(i => i.total)
        countryChart.update()
    }
    if (data.browser_stats) {
        browserChart.data.labels             = data.browser_stats.map(i => i.dimension_value)
        browserChart.data.datasets[0].data   = data.browser_stats.map(i => i.total)
        browserChart.update()
    }
}

async function fetchGeoStats() {
    try {
        const response = await fetch(props.config.routes.geoStats)
        if (!response.ok) throw new Error('Failed to fetch geolocation stats')
        const data = await response.json()
        Object.assign(geoStats, data)
    } catch (error) {
        console.error('Error fetching geolocation stats:', error)
    }
}

// ── Actions ───────────────────────────────────────────────────────────────────

function onDateRangeChange() {
    if (dateRange.value !== 'custom') fetchData()
}

async function clearGeoCache() {
    clearingCache.value = true
    try {
        const response = await fetch(props.config.routes.clearCache, {
            method: 'POST',
            headers: {
                'X-CSRF-TOKEN': csrfToken(),
                'Content-Type': 'application/json',
                'Accept': 'application/json',
            },
        })
        if (!response.ok) throw new Error('Failed to clear cache')
        await fetchGeoStats()
    } catch (error) {
        console.error('Error clearing geolocation cache:', error)
    } finally {
        setTimeout(() => { clearingCache.value = false }, 2000)
    }
}

function exportData() {
    const params = new URLSearchParams({ range: dateRange.value })
    if (dateRange.value === 'custom' && startDate.value && endDate.value) {
        params.append('start_date', startDate.value)
        params.append('end_date', endDate.value)
    }
    window.location.href = `${props.config.routes.export}?${params}`
}

// ── Lifecycle ─────────────────────────────────────────────────────────────────

onMounted(() => {
    initCharts()
    fetchData()
    refreshTimer = setInterval(fetchData, props.config.refreshInterval * 1000)
})

onUnmounted(() => {
    if (refreshTimer) clearInterval(refreshTimer)
    pageViewsChart?.destroy()
    deviceChart?.destroy()
    countryChart?.destroy()
    browserChart?.destroy()
})
</script>
