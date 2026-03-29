<template>
    <BaseTab tab-name="aerotune">
        <!-- Header -->
        <div class="aerotune-header">
            <div>
                <p class="aerotune-title">AeroTune™</p>
                <p class="aerotune-subtitle">Intelligent PID Calculator &amp; Blackbox Analyzer</p>
            </div>
        </div>

        <!-- Sub-tab navigation -->
        <div class="aerotune-tabs">
            <button
                class="at-tab-btn"
                :class="{ active: activeView === 'calculator' }"
                @click="activeView = 'calculator'"
            >
                STEP 1: CALCULATOR
            </button>
            <button class="at-tab-btn" :class="{ active: activeView === 'analyzer' }" @click="activeView = 'analyzer'">
                STEP 2: LOG ANALYZER
            </button>
            <button class="at-tab-btn" :class="{ active: activeView === 'autotune' }" @click="activeView = 'autotune'">
                AUTO TUNE
            </button>
            <button
                class="at-tab-btn"
                :class="{ active: activeView === 'instructions' }"
                @click="activeView = 'instructions'"
            >
                INSTRUCTIONS
            </button>
        </div>

        <div class="aerotune-body">
            <!-- ═══════════════ CALCULATOR ═══════════════ -->
            <div v-show="activeView === 'calculator'" class="at-view">
                <div class="at-calculator">
                    <!-- Left: Inputs -->
                    <div class="at-panel">
                        <div class="at-panel-header">⚙ SPECIFICATION INPUT</div>
                        <div class="at-panel-body">
                            <div class="at-form-row">
                                <label>Motor KV (200 – 11500)</label>
                                <input type="number" v-model.number="kv" min="200" max="11500" step="100" />
                            </div>

                            <div class="at-form-row">
                                <label>Battery Voltage</label>
                                <div class="at-voltage-presets">
                                    <button
                                        v-for="preset in voltagePresets"
                                        :key="preset.label"
                                        :class="{ selected: voltage === preset.v }"
                                        @click="selectVoltage(preset.v)"
                                    >
                                        {{ preset.label }}
                                    </button>
                                </div>
                                <input
                                    type="number"
                                    v-model.number="voltage"
                                    min="3.0"
                                    max="60.0"
                                    step="0.1"
                                    @input="voltageSelectedPreset = null"
                                />
                            </div>

                            <div class="at-form-row">
                                <label>Prop Size (inches, 2 – 11)</label>
                                <input type="number" v-model.number="prop" min="2" max="11" step="0.5" />
                            </div>

                            <div class="at-form-row">
                                <label>Total Weight g (80 – 5000)</label>
                                <input type="number" v-model.number="weight" min="80" max="5000" step="10" />
                            </div>

                            <div class="at-form-row">
                                <label>Flying Style</label>
                                <select v-model="style">
                                    <option value="Bando">Bando</option>
                                    <option value="Racing">Racing</option>
                                    <option value="Long Range">Long Range</option>
                                    <option value="Cinematic">Cinematic</option>
                                </select>
                            </div>

                            <button class="at-calc-btn" @click="calculate">🚀 CALCULATE PIDs</button>
                        </div>
                    </div>

                    <!-- Right: Outputs -->
                    <div class="at-panel">
                        <div class="at-panel-header">📊 CALCULATED PID OUTPUT</div>
                        <div class="at-panel-body">
                            <div class="at-no-result" v-if="!showResults">Enter specs and click CALCULATE PIDs</div>

                            <div v-else>
                                <table class="at-pid-table">
                                    <thead>
                                        <tr>
                                            <th class="at-pid-th-axis"></th>
                                            <th>
                                                Proportional
                                                <span
                                                    class="at-tip"
                                                    @mouseenter="
                                                        showTip(
                                                            $event,
                                                            'How far the drone reacts to an input — like how far a seesaw swings. Too low and it feels sluggish, too high and it overshoots.',
                                                        )
                                                    "
                                                    @mouseleave="hideTip"
                                                    >ⓘ</span
                                                >
                                            </th>
                                            <th>
                                                Integral
                                                <span
                                                    class="at-tip"
                                                    @mouseenter="
                                                        showTip(
                                                            $event,
                                                            'How quickly it returns to centre after a disturbance — like the seesaw finding balance. Too low and it drifts, too high and it hunts.',
                                                        )
                                                    "
                                                    @mouseleave="hideTip"
                                                    >ⓘ</span
                                                >
                                            </th>
                                            <th>
                                                Derivative
                                                <span
                                                    class="at-tip"
                                                    @mouseenter="
                                                        showTip(
                                                            $event,
                                                            'The dampening that cushions the movement — like a rubber tyre under the seesaw. Stops it bouncing back and forth after each input.',
                                                        )
                                                    "
                                                    @mouseleave="hideTip"
                                                    >ⓘ</span
                                                >
                                            </th>
                                            <th>
                                                D Max
                                                <span
                                                    class="at-tip"
                                                    @mouseenter="
                                                        showTip(
                                                            $event,
                                                            'The maximum dampening allowed at high throttle — D rises up to this ceiling during fast manoeuvres.',
                                                        )
                                                    "
                                                    @mouseleave="hideTip"
                                                    >ⓘ</span
                                                >
                                            </th>
                                            <th>
                                                Feedforward
                                                <span
                                                    class="at-tip"
                                                    @mouseenter="
                                                        showTip(
                                                            $event,
                                                            'How eagerly it anticipates your stick input — jumps ahead of the move rather than reacting to it.',
                                                        )
                                                    "
                                                    @mouseleave="hideTip"
                                                    >ⓘ</span
                                                >
                                            </th>
                                        </tr>
                                    </thead>
                                    <tbody>
                                        <tr class="at-pid-row at-pid-row--roll">
                                            <td class="at-pid-axis-label">ROLL</td>
                                            <td class="at-pid-num">{{ pids.roll_p }}</td>
                                            <td class="at-pid-num">{{ pids.roll_i }}</td>
                                            <td class="at-pid-num">{{ pids.d_min_roll }}</td>
                                            <td class="at-pid-num">{{ pids.dMax_roll }}</td>
                                            <td class="at-pid-num">{{ pids.roll_f }}</td>
                                        </tr>
                                        <tr class="at-pid-row at-pid-row--pitch">
                                            <td class="at-pid-axis-label">PITCH</td>
                                            <td class="at-pid-num">{{ pids.pitch_p }}</td>
                                            <td class="at-pid-num">{{ pids.pitch_i }}</td>
                                            <td class="at-pid-num">{{ pids.d_min_pitch }}</td>
                                            <td class="at-pid-num">{{ pids.dMax_pitch }}</td>
                                            <td class="at-pid-num">{{ pids.pitch_f }}</td>
                                        </tr>
                                        <tr class="at-pid-row at-pid-row--yaw">
                                            <td class="at-pid-axis-label">YAW</td>
                                            <td class="at-pid-num">{{ pids.yaw_p }}</td>
                                            <td class="at-pid-num">{{ pids.yaw_i }}</td>
                                            <td class="at-pid-num at-pid-num--muted">–</td>
                                            <td class="at-pid-num">{{ pids.yaw_d }}</td>
                                            <td class="at-pid-num">{{ pids.yaw_f }}</td>
                                        </tr>
                                    </tbody>
                                </table>

                                <div class="at-dmin-row">
                                    D_Min: &nbsp; Roll <span>{{ pids.d_min_roll }}</span> &nbsp;&nbsp; Pitch
                                    <span>{{ pids.d_min_pitch }}</span>
                                </div>

                                <div class="at-filter-rec">
                                    <div style="font-size: 11px; color: var(--subtleText); margin-bottom: 4px">
                                        RECOMMENDED GYRO LOWPASS 2
                                    </div>
                                    <div>
                                        <span class="at-filter-val">{{ filterRec.hz }}</span>
                                        <span style="font-size: 11px; color: var(--subtleText)">
                                            Hz &nbsp;·&nbsp;
                                        </span>
                                        <span style="font-size: 11px; color: var(--subtleText)">{{
                                            filterRec.note
                                        }}</span>
                                    </div>
                                    <div style="font-size: 11px; color: var(--subtleText); margin-top: 4px">
                                        Range: {{ filterRec.low }} – {{ filterRec.high }} Hz
                                    </div>
                                </div>

                                <button class="at-apply-btn" :disabled="!canApply" @click="applyToFC">
                                    ✔ APPLY PIDs TO FC (PID TUNING TAB)
                                </button>
                                <button class="at-copy-btn" @click="copyValues">{{ copyBtnText }}</button>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- ═══════════════ ANALYZER ═══════════════ -->
            <div v-show="activeView === 'analyzer'" class="at-view">
                <div class="at-analyzer">
                    <div class="at-panel" style="margin-bottom: 12px">
                        <div class="at-panel-header">WORKFLOW</div>
                        <div class="at-panel-body" style="font-size: 12px; color: var(--subtleText); line-height: 1.8">
                            {{ workflowInstructions }}
                        </div>
                    </div>

                    <div class="at-panel">
                        <div class="at-panel-header">📂 FLIGHT LOG ANALYSIS (HIGH THROTTLE)</div>
                        <div class="at-panel-body">
                            <div class="at-form-row" style="margin-bottom: 10px">
                                <label style="font-size: 12px; color: var(--subtleText)">Motors after flight:</label>
                                <div class="at-voltage-presets">
                                    <button
                                        v-for="t in ['COOL', 'WARM', 'HOT']"
                                        :key="t"
                                        :class="{ selected: motorTemp === t }"
                                        @click="motorTemp = t"
                                    >
                                        {{ t }}
                                    </button>
                                </div>
                            </div>

                            <div class="at-file-row">
                                <button type="button" class="at-file-label" @click="$refs.fileInput.click()">
                                    Select BBL / BFL / CSV
                                </button>
                                <input
                                    type="file"
                                    ref="fileInput"
                                    accept=".bfl,.bbl,.csv"
                                    style="display: none"
                                    @change="onFileChange"
                                />
                                <span class="at-file-name">{{ fileName }}</span>
                                <button id="at-analyze-btn" :disabled="!csvFile" @click="analyzeFile">
                                    🔍 ANALYZE
                                </button>
                            </div>

                            <div v-if="bblSessions.length > 1" class="at-form-row" style="margin-top: 8px">
                                <label style="font-size: 12px; color: var(--subtleText)">Select flight session:</label>
                                <select v-model.number="bblSelectedSession" @change="runBBLSession(bblSelectedSession)">
                                    <option v-for="(_, idx) in bblSessions" :key="idx" :value="idx">
                                        Session {{ idx + 1 }}
                                    </option>
                                </select>
                            </div>

                            <!-- ═══ FLIGHT DATA GRAPHS ═══ -->
                            <div v-if="graphsVisible" class="at-graphs-section" id="at-graphs">
                                <!-- Graph 1: Unfiltered Gyros -->
                                <div class="at-graph-panel">
                                    <div class="at-graph-header">
                                        <span class="at-graph-title">UNFILTERED GYRO</span>
                                        <div class="at-graph-toggles">
                                            <button
                                                v-for="ax in graphAxes"
                                                :key="'g1-' + ax.name"
                                                :class="['at-axis-toggle', { active: graphToggles.gyro[ax.name] }]"
                                                :style="{ '--ax-color': ax.color }"
                                                @click="
                                                    graphToggles.gyro[ax.name] = !graphToggles.gyro[ax.name];
                                                    renderGraphs();
                                                "
                                            >
                                                {{ ax.label }}
                                            </button>
                                        </div>
                                        <div class="at-graph-zoom">
                                            <button @click="graphZoom('gyro', -1)">−</button>
                                            <button @click="graphZoom('gyro', 0)">Reset</button>
                                            <button @click="graphZoom('gyro', 1)">+</button>
                                        </div>
                                    </div>
                                    <canvas ref="graphGyro" class="at-graph-canvas" width="900" height="180"></canvas>
                                </div>

                                <!-- Graph 2: Setpoint + Gyro Tracking -->
                                <div class="at-graph-panel">
                                    <div class="at-graph-header">
                                        <span class="at-graph-title">Setpoint vs Gyro</span>
                                        <div class="at-graph-toggles">
                                            <button
                                                v-for="ax in graphAxes"
                                                :key="'g2-' + ax.name"
                                                :class="['at-axis-toggle', { active: graphToggles.setpoint[ax.name] }]"
                                                :style="{ '--ax-color': ax.color }"
                                                @click="
                                                    graphToggles.setpoint[ax.name] = !graphToggles.setpoint[ax.name];
                                                    renderGraphs();
                                                "
                                            >
                                                {{ ax.label }}
                                            </button>
                                        </div>
                                        <div class="at-graph-zoom">
                                            <button @click="graphZoom('setpoint', -1)">-</button>
                                            <button @click="graphZoom('setpoint', 0)">Fit</button>
                                            <button @click="graphZoom('setpoint', 1)">+</button>
                                        </div>
                                    </div>
                                    <canvas
                                        ref="graphSetpoint"
                                        class="at-graph-canvas"
                                        width="900"
                                        height="180"
                                    ></canvas>
                                </div>

                                <!-- Graph 3: PID Error -->
                                <div class="at-graph-panel">
                                    <div class="at-graph-header">
                                        <span class="at-graph-title">PID ERROR (GYRO − SETPOINT)</span>
                                        <div class="at-graph-toggles">
                                            <button
                                                v-for="ax in graphAxes"
                                                :key="'g3-' + ax.name"
                                                :class="['at-axis-toggle', { active: graphToggles.pidError[ax.name] }]"
                                                :style="{ '--ax-color': ax.color }"
                                                @click="
                                                    graphToggles.pidError[ax.name] = !graphToggles.pidError[ax.name];
                                                    renderGraphs();
                                                "
                                            >
                                                {{ ax.label }}
                                            </button>
                                        </div>
                                        <div class="at-graph-zoom">
                                            <button @click="graphZoom('pidError', -1)">−</button>
                                            <button @click="graphZoom('pidError', 0)">Reset</button>
                                            <button @click="graphZoom('pidError', 1)">+</button>
                                        </div>
                                    </div>
                                    <canvas
                                        ref="graphPidError"
                                        class="at-graph-canvas"
                                        width="900"
                                        height="180"
                                    ></canvas>
                                </div>

                                <!-- Graph 4: Freq vs Throttle Spectrogram -->
                                <div class="at-graph-panel">
                                    <div class="at-graph-header">
                                        <span class="at-graph-title">Frequency vs Throttle</span>
                                        <div class="at-spectrogram-legend">
                                            <span class="at-sg-quiet">quiet</span>
                                            <canvas ref="spectrogramLegend" width="100" height="10"></canvas>
                                            <span class="at-sg-loud">loud</span>
                                        </div>
                                    </div>
                                    <div class="at-spectrogram-row">
                                        <canvas
                                            ref="graphSpectrogram"
                                            class="at-graph-canvas at-graph-canvas--tall"
                                            width="900"
                                            height="280"
                                        ></canvas>
                                        <input
                                            type="range"
                                            class="at-gain-slider"
                                            min="10"
                                            max="400"
                                            :value="spectrogramGain"
                                            orient="vertical"
                                            title="Intensity gain"
                                            @input="
                                                spectrogramGain = Number($event.target.value);
                                                renderGraphs();
                                            "
                                        />
                                    </div>
                                </div>

                                <!-- Motor RPM Heatmap -->
                                <div class="at-graph-panel" v-if="extendedAnalysis && extendedAnalysis.motorHeatmap">
                                    <div class="at-graph-header">
                                        <span class="at-graph-title">Motor RPM Heatmap</span>
                                        <span class="at-graph-subtitle" v-if="extendedAnalysis.itermBiasShort">{{
                                            extendedAnalysis.itermBiasShort
                                        }}</span>
                                    </div>
                                    <canvas
                                        ref="graphMotorHeat"
                                        class="at-graph-canvas"
                                        width="900"
                                        height="140"
                                    ></canvas>
                                </div>
                            </div>

                            <!-- ═══ ANALYSIS RESULTS ═══ -->
                            <div class="at-results-box">{{ analysisResult }}</div>

                            <!-- ═══ Extended Analysis (text) ═══ -->
                            <div v-if="extendedAnalysis" class="at-extended-analysis">
                                <div class="at-ext-section">
                                    <div class="at-ext-header">Step Response</div>
                                    <div class="at-ext-body at-mono">{{ extendedAnalysis.stepResponseText }}</div>
                                </div>
                            </div>

                            <!-- ═══ PID Output — Old vs New ═══ -->
                            <div v-if="logPidOutput" class="at-log-pid-section">
                                <div class="at-ext-header">PID RECOMMENDATIONS — OLD vs NEW</div>
                                <table class="at-pid-table at-log-pid-table">
                                    <thead>
                                        <tr>
                                            <th></th>
                                            <th colspan="3">OLD (from BBL)</th>
                                            <th colspan="3">NEW (recommended)</th>
                                        </tr>
                                        <tr>
                                            <th></th>
                                            <th>P</th>
                                            <th>I</th>
                                            <th>D</th>
                                            <th>P</th>
                                            <th>I</th>
                                            <th>D</th>
                                        </tr>
                                    </thead>
                                    <tbody>
                                        <tr
                                            v-for="ax in ['roll', 'pitch', 'yaw']"
                                            :key="'pid-' + ax"
                                            :class="'at-pid-row at-pid-row--' + ax"
                                        >
                                            <td class="at-pid-axis-label">{{ ax.toUpperCase() }}</td>
                                            <td class="at-pid-num">{{ logPidOutput.old[ax].P }}</td>
                                            <td class="at-pid-num">{{ logPidOutput.old[ax].I }}</td>
                                            <td class="at-pid-num">{{ logPidOutput.old[ax].D }}</td>
                                            <td class="at-pid-num at-pid-num--new">{{ logPidOutput.new[ax].P }}</td>
                                            <td class="at-pid-num at-pid-num--new">{{ logPidOutput.new[ax].I }}</td>
                                            <td class="at-pid-num at-pid-num--new">{{ logPidOutput.new[ax].D }}</td>
                                        </tr>
                                    </tbody>
                                </table>
                                <div class="at-pid-actions">
                                    <button class="at-apply-btn" :disabled="!canApplyLogPids" @click="applyLogPidsToFC">
                                        ✓ APPLY NEW PIDs TO FC
                                    </button>
                                    <button class="at-copy-btn" @click="copyLogPids">{{ logPidCopyBtnText }}</button>
                                </div>
                            </div>

                            <!-- ═══ SysID / Chirp frequency-response results ═══ -->
                            <div v-if="sysidResult" class="at-sysid-section">
                                <div class="at-sysid-banner">
                                    ⚡ CHIRP / SYSID LOG DETECTED — running frequency response analysis
                                </div>

                                <!-- Axis selector tabs -->
                                <div class="at-sysid-axis-tabs">
                                    <button
                                        v-for="ax in sysidAvailableAxes"
                                        :key="ax.name"
                                        class="at-sysid-axis-tab"
                                        :class="{ active: sysidActiveAxis === ax.name }"
                                        :style="{ '--ax-color': ax.color }"
                                        @click="selectSysIDAxis(ax.name)"
                                    >
                                        {{ ax.label }}
                                    </button>
                                </div>

                                <!-- Single setpoint vs gyro overlay chart -->
                                <canvas
                                    ref="chirpOverlayCanvas"
                                    class="at-chirp-overlay-canvas"
                                    width="580"
                                    height="260"
                                ></canvas>

                                <!-- Chart legend + hint -->
                                <div class="at-chirp-overlay-legend">
                                    Coloured line = setpoint (command). White line = gyro (actual response).
                                </div>
                                <div class="at-chirp-overlay-hint">
                                    Where the white trace stops following the coloured trace is your bandwidth limit.
                                </div>

                                <!-- Zoom presets -->
                                <div class="at-sysid-zoom-row">
                                    <button
                                        :class="['at-sysid-zoom-btn', { active: sysidZoom === 'full' }]"
                                        @click="setSysIDZoom('full')"
                                    >
                                        Full sweep
                                    </button>
                                    <button
                                        :class="['at-sysid-zoom-btn', { active: sysidZoom === 'zoomed' }]"
                                        @click="setSysIDZoom('zoomed')"
                                    >
                                        Zoomed (middle 50%)
                                    </button>
                                </div>

                                <!-- Frequency Response PID Output Table -->
                                <div v-if="sysidPids" class="at-sysid-pid-section at-sysid-pid-output">
                                    <div class="at-sysid-pid-header at-sysid-pid-complete-header">
                                        ✓ FREQUENCY RESPONSE ANALYSIS COMPLETE
                                    </div>
                                    <table class="at-pid-table at-sysid-pid-table">
                                        <thead>
                                            <tr>
                                                <th class="at-pid-th-axis"></th>
                                                <th>PROPORTIONAL</th>
                                                <th>INTEGRAL</th>
                                                <th>DERIVATIVE</th>
                                                <th>D MAX</th>
                                                <th>FEEDFORWARD</th>
                                            </tr>
                                        </thead>
                                        <tbody>
                                            <tr class="at-pid-row at-pid-row--roll">
                                                <td class="at-pid-axis-label">ROLL</td>
                                                <td class="at-pid-num">{{ sysidPids.roll.P }}</td>
                                                <td class="at-pid-num">{{ sysidPids.roll.I }}</td>
                                                <td class="at-pid-num">{{ sysidPids.roll.D }}</td>
                                                <td class="at-pid-num">{{ sysidPids.roll.DMax }}</td>
                                                <td class="at-pid-num">{{ sysidPids.roll.FF }}</td>
                                            </tr>
                                            <tr class="at-pid-row at-pid-row--pitch">
                                                <td class="at-pid-axis-label">PITCH</td>
                                                <td class="at-pid-num">{{ sysidPids.pitch.P }}</td>
                                                <td class="at-pid-num">{{ sysidPids.pitch.I }}</td>
                                                <td class="at-pid-num">{{ sysidPids.pitch.D }}</td>
                                                <td class="at-pid-num">{{ sysidPids.pitch.DMax }}</td>
                                                <td class="at-pid-num">{{ sysidPids.pitch.FF }}</td>
                                            </tr>
                                            <tr class="at-pid-row at-pid-row--yaw">
                                                <td class="at-pid-axis-label">YAW</td>
                                                <td class="at-pid-num">{{ sysidPids.yaw.P }}</td>
                                                <td class="at-pid-num">{{ sysidPids.yaw.I }}</td>
                                                <td class="at-pid-num at-pid-num--muted">–</td>
                                                <td class="at-pid-num">{{ sysidPids.yaw.DMax }}</td>
                                                <td class="at-pid-num">{{ sysidPids.yaw.FF }}</td>
                                            </tr>
                                        </tbody>
                                    </table>
                                    <div class="at-sysid-pid-note" style="margin-top: 8px">
                                        Hover-condition baseline — I and FF unchanged. Re-fly with aggressive data to
                                        fine-tune further.
                                    </div>
                                    <div
                                        v-if="sysidPids.roll.reason || sysidPids.pitch.reason"
                                        class="at-sysid-pid-note"
                                        style="margin-top: 4px; font-size: 11px; color: var(--subtleText)"
                                    >
                                        <span v-if="sysidPids.roll.reason">Roll: ({{ sysidPids.roll.reason }})</span>
                                        <span v-if="sysidPids.roll.reason && sysidPids.pitch.reason"> · </span>
                                        <span v-if="sysidPids.pitch.reason">Pitch: ({{ sysidPids.pitch.reason }})</span>
                                    </div>
                                    <button class="at-apply-btn" :disabled="!canApplySysID" @click="applySysIDToFC">
                                        ✓ APPLY PIDs TO FC (PID TUNING TAB)
                                    </button>
                                    <button class="at-copy-btn" @click="copySysIDValues">{{ sysidCopyBtnText }}</button>
                                </div>

                                <!-- Warnings -->
                                <div v-if="sysidResult.warnings.length > 0" class="at-sysid-warnings">
                                    <div v-for="(w, i) in sysidResult.warnings" :key="i" class="at-sysid-warning-item">
                                        ⚠ {{ w }}
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- ═══════════════ INSTRUCTIONS ═══════════════ -->
            <div v-show="activeView === 'instructions'" class="at-view">
                <div class="at-instructions">
                    <button class="at-instructions-popup-btn" @click="openInstructionsPopup">
                        ↗ Open in New Window
                    </button>
                    <h3>STEP 1: CALCULATE BASELINE PIDs</h3>
                    <ul>
                        <li>Enter motor KV, battery voltage, prop size, weight and flying style.</li>
                        <li>Click <strong>CALCULATE PIDs</strong> to get conservative baseline values.</li>
                        <li>
                            Click <strong>APPLY PIDs TO FC</strong> to write them directly to the PID Tuning tab, or use
                            <strong>COPY ALL VALUES</strong> to copy them to the clipboard.
                        </li>
                    </ul>

                    <h3>STEP 2: CONFIGURE BLACKBOX</h3>
                    <ul>
                        <li>In Betaflight, go to Configuration → Blackbox → Enable, Device = SD Card.</li>
                        <li>
                            Set Blackbox logging rate to 1/2 or better — higher rates give better frequency resolution
                            for the analyzer.
                        </li>
                        <li>Enable: Gyro, Gyro (Unfiltered), Motor, PID, RC Commands, RPM, Setpoint, Accelerometer.</li>
                        <li><strong>Betaflight 4.5+:</strong> raw gyro is always logged automatically.</li>
                        <li>
                            <strong>Betaflight 4.3/4.4:</strong> set Debug Mode to <code>GYRO_SCALED</code> to capture
                            unfiltered gyro data.
                        </li>
                        <li>
                            Use fresh propellers — damaged props introduce false noise and will give inaccurate results.
                        </li>
                    </ul>

                    <h3>STEP 3: FLY THE TEST PATTERN</h3>
                    <ul>
                        <li>Level mode or Acro mode both work — LOS or FPV.</li>
                        <li>
                            Level mode: full left stick hold 1–1.5 seconds, pause, full right, pause, full forward,
                            pause, full back.
                        </li>
                        <li>Acro mode: sharp direct inputs at 20° and 45°, with brief pauses between each.</li>
                        <li>
                            Aim for a 2 minute flight. Fly through the full throttle range — the Analyzer needs data
                            across all throttle levels to give an accurate result.
                        </li>
                    </ul>

                    <h3>STEP 4: ANALYZE THE LOG</h3>
                    <ul>
                        <li>In Betaflight, go to the Blackbox tab and click USB Storage Mode.</li>
                        <li>Drag your .bfl file from the FC storage to your desktop.</li>
                        <li>Unplug the FC, then plug it back in and open Betaflight.</li>
                        <li>In the AeroTune tab, click Select BBL / BFL, choose your .bfl file and click ANALYZE.</li>
                    </ul>

                    <h3>INTERPRETING RESULTS</h3>
                    <ul>
                        <li>
                            <span class="at-status-ok">EXCELLENT / CLEAN</span> – filters are well-tuned, no changes
                            needed
                        </li>
                        <li><span class="at-status-ok">GOOD</span> – minor adjustments may help</li>
                        <li><span class="at-status-warn">FAIR</span> – lower Gyro Lowpass 2 by ~30 Hz, re-test</li>
                        <li>
                            <span class="at-status-warn">WEAK</span> – lower by ~50 Hz, consider adding a Notch filter
                        </li>
                        <li>
                            <span class="at-status-bad">VERY WEAK</span> – aggressive filter reduction needed; check for
                            mechanical vibration
                        </li>
                    </ul>

                    <h3>FILTER RECOMMENDATION NOTE</h3>
                    <ul>
                        <li>
                            <strong>Gyro Lowpass 2:</strong> The value shown in the Calculator is a
                            <em>starting point</em> based on prop size. Use the Analyzer results to fine-tune after
                            flying.
                        </li>
                        <li>
                            <strong>Dynamic Notch Filter:</strong> Enable and adjust count based on analyzer results to
                            target resonant frequencies.
                        </li>
                        <li>
                            <strong>Gyro RPM Filter:</strong> Enable if using bidirectional DSHOT — the most effective
                            filter available for eliminating motor noise harmonics.
                        </li>
                    </ul>
                </div>
            </div>

            <!-- ═══════════════ AUTO TUNE ═══════════════ -->
            <div v-show="activeView === 'autotune'" class="at-view">
                <!-- Prop size selector -->
                <div class="at-panel at-prop-selector">
                    <div class="at-panel-header">
                        PROP SIZE
                        <span
                            class="at-tip"
                            @mouseenter="
                                showTip(
                                    $event,
                                    'Select your prop diameter. This sets smart defaults for sweep frequency range and shake amplitude. You can still override them in Advanced Settings.',
                                )
                            "
                            @mouseleave="hideTip"
                            >ⓘ</span
                        >
                    </div>
                    <div class="at-panel-body">
                        <div class="at-prop-btns">
                            <button
                                v-for="size in [3, 4, 5, 6, 7, 8, 9, 10]"
                                :key="size"
                                :class="['at-prop-btn', { active: chirpPropInch === size }]"
                                @click="chirpPropInch = size"
                            >
                                {{ size }}"
                            </button>
                        </div>
                    </div>
                </div>

                <!-- Axis amplitude cards -->
                <div class="at-chirp-cards">
                    <div class="at-panel at-chirp-card">
                        <div class="at-panel-header at-chirp-card-header--pitch">PITCH</div>
                        <div class="at-panel-body">
                            <div class="at-intensity-label">
                                SHAKE INTENSITY
                                <span
                                    class="at-tip"
                                    @mouseenter="
                                        showTip(
                                            $event,
                                            'How hard we shake the drone to measure its response. Like a firm handshake — harder means more data but more movement. Start with MEDIUM.',
                                        )
                                    "
                                    @mouseleave="hideTip"
                                    >ⓘ</span
                                >
                            </div>
                            <div class="at-intensity-btns">
                                <button
                                    :class="['at-intensity-btn', { active: chirpPitchLevel === 'EASY' }]"
                                    @click="setChirpLevel('pitch', 'EASY')"
                                >
                                    EASY
                                </button>
                                <button
                                    :class="['at-intensity-btn', { active: chirpPitchLevel === 'MEDIUM' }]"
                                    @click="setChirpLevel('pitch', 'MEDIUM')"
                                >
                                    MEDIUM
                                </button>
                                <button
                                    :class="['at-intensity-btn', { active: chirpPitchLevel === 'HARD' }]"
                                    @click="setChirpLevel('pitch', 'HARD')"
                                >
                                    HARD
                                </button>
                            </div>
                        </div>
                    </div>
                    <div class="at-panel at-chirp-card">
                        <div class="at-panel-header at-chirp-card-header--roll">ROLL</div>
                        <div class="at-panel-body">
                            <div class="at-intensity-label">
                                SHAKE INTENSITY
                                <span
                                    class="at-tip"
                                    @mouseenter="
                                        showTip(
                                            $event,
                                            'How hard we shake the drone to measure its response. Like a firm handshake — harder means more data but more movement. Start with MEDIUM.',
                                        )
                                    "
                                    @mouseleave="hideTip"
                                    >ⓘ</span
                                >
                            </div>
                            <div class="at-intensity-btns">
                                <button
                                    :class="['at-intensity-btn', { active: chirpRollLevel === 'EASY' }]"
                                    @click="setChirpLevel('roll', 'EASY')"
                                >
                                    EASY
                                </button>
                                <button
                                    :class="['at-intensity-btn', { active: chirpRollLevel === 'MEDIUM' }]"
                                    @click="setChirpLevel('roll', 'MEDIUM')"
                                >
                                    MEDIUM
                                </button>
                                <button
                                    :class="['at-intensity-btn', { active: chirpRollLevel === 'HARD' }]"
                                    @click="setChirpLevel('roll', 'HARD')"
                                >
                                    HARD
                                </button>
                            </div>
                        </div>
                    </div>
                    <div class="at-panel at-chirp-card">
                        <div class="at-panel-header at-chirp-card-header--yaw">YAW</div>
                        <div class="at-panel-body">
                            <div class="at-intensity-label">
                                SHAKE INTENSITY
                                <span
                                    class="at-tip"
                                    @mouseenter="
                                        showTip(
                                            $event,
                                            'How hard we shake the drone to measure its response. Like a firm handshake — harder means more data but more movement. Start with MEDIUM.',
                                        )
                                    "
                                    @mouseleave="hideTip"
                                    >ⓘ</span
                                >
                            </div>
                            <div class="at-intensity-btns">
                                <button
                                    :class="['at-intensity-btn', { active: chirpYawLevel === 'EASY' }]"
                                    @click="setChirpLevel('yaw', 'EASY')"
                                >
                                    EASY
                                </button>
                                <button
                                    :class="['at-intensity-btn', { active: chirpYawLevel === 'MEDIUM' }]"
                                    @click="setChirpLevel('yaw', 'MEDIUM')"
                                >
                                    MEDIUM
                                </button>
                                <button
                                    :class="['at-intensity-btn', { active: chirpYawLevel === 'HARD' }]"
                                    @click="setChirpLevel('yaw', 'HARD')"
                                >
                                    HARD
                                </button>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- Configure button -->
                <button class="at-chirp-configure-btn" @click="configureFc">⚙ CONFIGURE ALL AXES ON FC</button>

                <!-- Confirmation box -->
                <div v-if="chirpConfigured" class="at-chirp-confirm">
                    <div class="at-chirp-confirm-title">✓ Commands sent to FC:</div>
                    <pre>{{ chirpConfirmText }}</pre>
                </div>

                <!-- Advanced settings (collapsed by default) -->
                <div class="at-advanced-section">
                    <button
                        type="button"
                        class="at-advanced-toggle"
                        @click="advancedOpen = !advancedOpen"
                        :aria-expanded="advancedOpen"
                    >
                        ⚙ ADVANCED SETTINGS
                        <span class="at-advanced-chevron" :class="{ open: advancedOpen }">▶</span>
                    </button>
                    <div v-if="advancedOpen" class="at-advanced-body">
                        <div class="at-form-row">
                            <label
                                >Sweep Start Hz
                                <span
                                    class="at-tip"
                                    @mouseenter="
                                        showTip(
                                            $event,
                                            'The lowest frequency in the sweep. Leave at default unless you know your problem frequency.',
                                        )
                                    "
                                    @mouseleave="hideTip"
                                    >ⓘ</span
                                ></label
                            >
                            <input type="number" v-model.number="chirpStartHz" min="0.1" max="100" step="0.1" />
                        </div>
                        <div class="at-form-row">
                            <label
                                >Sweep End Hz
                                <span
                                    class="at-tip"
                                    @mouseenter="
                                        showTip(
                                            $event,
                                            'The highest frequency in the sweep. 600Hz covers all relevant drone frequencies.',
                                        )
                                    "
                                    @mouseleave="hideTip"
                                    >ⓘ</span
                                ></label
                            >
                            <input type="number" v-model.number="chirpEndHz" min="1" max="1000" step="1" />
                        </div>
                        <div class="at-form-row">
                            <label
                                >Duration seconds (1 – 60)
                                <span
                                    class="at-tip"
                                    @mouseenter="
                                        showTip(
                                            $event,
                                            'How long the chirp runs per axis. Longer means more accurate data. 20 seconds is ideal.',
                                        )
                                    "
                                    @mouseleave="hideTip"
                                    >ⓘ</span
                                ></label
                            >
                            <input type="number" v-model.number="chirpDuration" min="1" max="60" step="1" />
                        </div>
                    </div>
                </div>

                <!-- Flight procedure instructions -->
                <div class="at-panel at-chirp-instructions">
                    <div class="at-panel-header">📋 FLIGHT PROCEDURE</div>
                    <div class="at-panel-body">
                        <p style="color: #ffe66d; font-weight: bold; margin-bottom: 8px">
                            ⚠️ CAUTION: CHIRP IS EXPERIMENTAL
                        </p>
                        <p style="font-size: 12px; margin-bottom: 10px">
                            Fly in a large open area, maintain visual line of sight or fly FPV, be prepared to disarm
                            immediately, and fly at your own risk.
                        </p>
                        <ol>
                            <li>
                                Connected via USB — configure chirp settings above, hit button, then assign a dedicated
                                <code>CHIRP</code> switch in the Modes tab.
                            </li>
                            <li>Unplug USB and fly.</li>
                            <li>Hover to 5m or more — switch to Level mode if desired, or fly FPV.</li>
                            <li>
                                Flip the <code>CHIRP</code> switch once — firmware runs Pitch, Roll, then Yaw
                                automatically (~10 seconds per axis, ~30 seconds total).
                            </li>
                            <li>To abort: move sticks or toggle the <code>CHIRP</code> switch off.</li>
                            <li>
                                Land, plug in USB → load your .bfl file into
                                <strong>STEP 4: LOG ANALYZER</strong> — the analyzer will automatically detect the chirp
                                and run SysID analysis.
                            </li>
                        </ol>
                    </div>
                </div>
            </div>
        </div>
        <!-- /.aerotune-body --> </BaseTab
    ><!-- /.tab-aerotune -->

    <Teleport to="body">
        <div
            v-if="tooltip.visible"
            class="at-tooltip-bubble"
            :style="{ left: tooltip.x + 'px', top: tooltip.y + 'px' }"
        >
            {{ tooltip.text }}
        </div>
    </Teleport>
</template>

<script>
import FC from "@/js/fc";
import CONFIGURATOR from "@/js/data_storage";
import BaseTab from "./BaseTab.vue";
import MSP from "@/js/msp";
import MSPCodes from "@/js/msp/MSPCodes";
import { mspHelper } from "@/js/msp/MSPHelper";
import { usePidTuningStore } from "@/stores/pidTuning";
import { serial } from "@/js/serial";
import { BBLHeaderParser } from "@/js/aerotune/bbl-header-parser.js";
import { FrameDecoder } from "@/js/aerotune/frame-decoder.js";

// ─────────────────────────────────────────────────────────────────────────────
// AeroTune V5.6 Calculator
// ─────────────────────────────────────────────────────────────────────────────

const KV_BASELINE = {
    200: 38,
    1300: 54,
    1400: 61,
    1600: 73,
    1700: 75,
    1800: 77,
    2000: 82,
    2100: 88,
    2400: 84,
    3000: 77,
    3800: 73,
    4200: 78,
    5000: 67,
    7000: 54,
    11500: 90,
};

// Normalises KV_BASELINE so that 5"/4S 2000KV Racing at 500g → rollP ≈ 51
// (matches Betaflight 4.x defaults)
const BASE_NORM = 0.576;

const FLYING_STYLES = {
    Racing: 1.08,
    Bando: 1.0,
    "Long Range": 0.65,
    Cinematic: 0.75,
};

// FF baselines at 5"/4S (both scalars = 1.0); half-correction is applied on top
const FF_BY_STYLE = {
    Cinematic: { roll_f: 75, pitch_f: 80, yaw_f: 75 },
    "Long Range": { roll_f: 75, pitch_f: 80, yaw_f: 75 },
    Bando: { roll_f: 100, pitch_f: 105, yaw_f: 100 },
    Racing: { roll_f: 125, pitch_f: 131, yaw_f: 125 },
};

function interpolateKV(kv) {
    kv = Number.parseFloat(kv);
    const kvKeys = Object.keys(KV_BASELINE).map(Number);
    const minKey = Math.min(...kvKeys);
    const maxKey = Math.max(...kvKeys);
    kv = Math.max(minKey, Math.min(maxKey, kv));
    if (KV_BASELINE[kv] !== undefined) {
        return KV_BASELINE[kv];
    }
    const sorted = Object.keys(KV_BASELINE)
        .map(Number)
        .sort((a, b) => b - a);
    for (let i = 0; i < sorted.length - 1; i++) {
        const kv1 = sorted[i],
            kv2 = sorted[i + 1];
        if (kv <= kv1 && kv >= kv2) {
            const p1 = KV_BASELINE[kv1],
                p2 = KV_BASELINE[kv2];
            const ratio = kv1 !== kv2 ? (kv - kv2) / (kv1 - kv2) : 0;
            return p1 + (p2 - p1) * ratio;
        }
    }
    return KV_BASELINE[
        Object.keys(KV_BASELINE)
            .map(Number)
            .sort((a, b) => a - b)[0]
    ];
}

// Linear interpolation through a sorted [x, y] point table.
function interpolatePoints(x, points) {
    if (x <= points[0][0]) {
        return points[0][1];
    }
    if (x >= points[points.length - 1][0]) {
        return points[points.length - 1][1];
    }
    for (let i = 0; i < points.length - 1; i++) {
        if (x >= points[i][0] && x <= points[i + 1][0]) {
            const t = (x - points[i][0]) / (points[i + 1][0] - points[i][0]);
            return points[i][1] + t * (points[i + 1][1] - points[i][1]);
        }
    }
    return 1;
}

// Prop-size defaults for chirp sweep parameters.
// Each entry: [propInches, { startHz, endHz, easy, medium, hard }]
const CHIRP_PROP_DEFAULTS = [
    [3, { startHz: 100, endHz: 800, easy: 150, medium: 250, hard: 400 }],
    [5, { startHz: 80, endHz: 600, easy: 120, medium: 230, hard: 350 }],
    [7, { startHz: 50, endHz: 400, easy: 80, medium: 150, hard: 250 }],
    [10, { startHz: 30, endHz: 300, easy: 50, medium: 100, hard: 180 }],
];

function chirpDefaultsForProp(propInch) {
    const pts = CHIRP_PROP_DEFAULTS;
    if (propInch <= pts[0][0]) {
        return { ...pts[0][1] };
    }
    if (propInch >= pts[pts.length - 1][0]) {
        return { ...pts[pts.length - 1][1] };
    }
    for (let i = 0; i < pts.length - 1; i++) {
        const [x0, d0] = pts[i];
        const [x1, d1] = pts[i + 1];
        if (propInch >= x0 && propInch <= x1) {
            const t = (propInch - x0) / (x1 - x0);
            return {
                startHz: Math.round(d0.startHz + t * (d1.startHz - d0.startHz)),
                endHz: Math.round(d0.endHz + t * (d1.endHz - d0.endHz)),
                easy: Math.round(d0.easy + t * (d1.easy - d0.easy)),
                medium: Math.round(d0.medium + t * (d1.medium - d0.medium)),
                hard: Math.round(d0.hard + t * (d1.hard - d0.hard)),
            };
        }
    }
    return { ...pts[pts.length - 1][1] };
}

// Voltage scalar — 4S (14.8 V) is the baseline (1.00).
// Applied FULLY to P and D; HALF correction applied to I and FF.
function voltageScalar(voltage) {
    return interpolatePoints(Number.parseFloat(voltage), [
        [3.7, 1.1], // 1S
        [7.4, 1.1], // 2S
        [11.1, 1.1], // 3S
        [14.8, 1.0], // 4S  ← baseline
        [18.5, 0.95], // 5S
        [22.2, 0.87], // 6S
        [29.6, 0.77], // 8S
    ]);
}

// Prop size scalar — 5" is the baseline (1.00).
// Applied FULLY to P; HALF correction applied to I.
function propScalar(prop) {
    return interpolatePoints(Number.parseFloat(prop), [
        [2, 0.65],
        [3, 0.88],
        [3.5, 0.89],
        [4, 0.9],
        [5, 1.0], // ← baseline
        [6, 1.1],
        [7, 1.18],
        [8, 1.25],
    ]);
}

// D-term ratio — prop-size-aware. Smaller props need relatively higher D
// to damp faster oscillation modes. Calibrated: 3"/4S/Bando → D=35,
// 5" baseline → 0.61. d_min uses 0.887× this ratio.
function dRatio(prop) {
    return interpolatePoints(Number.parseFloat(prop), [
        [2, 0.95],
        [3, 0.84], // → D=35 at 3"/4S/Bando/2000KV anchor
        [3.5, 0.8],
        [4, 0.76], // → D=35 at 4"/4S/Racing/2000KV anchor
        [5, 0.61], // ← baseline
        [6, 0.55],
        [7, 0.5],
        [8, 0.46],
    ]);
}

function clamp(v, lo, hi) {
    return Math.max(lo, Math.min(hi, v));
}

function calculatePIDs(kv, voltage, prop, weight, style) {
    kv = Number.parseFloat(kv);
    voltage = Number.parseFloat(voltage);
    prop = Number.parseFloat(prop);
    weight = Number.parseFloat(weight);
    if (Number.isNaN(kv) || Number.isNaN(voltage) || Number.isNaN(prop) || Number.isNaN(weight)) {
        return null;
    }

    // Base P at 5"/4S scale — only KV, style, and weight contribute here.
    const rawBase = interpolateKV(kv) * (FLYING_STYLES[style] || 1) * (1 + ((weight - 500) / 2000) * 0.15);
    const base = rawBase * BASE_NORM; // normalised roll base

    // Voltage × prop correction
    const vScal = voltageScalar(voltage);
    const pScal = propScalar(prop);
    const dr = dRatio(prop);

    const fullMult = vScal * pScal; // P — full voltage+prop correction
    const halfMult = 1 + (fullMult - 1) * 0.5; // I — half voltage+prop correction
    const dMult = vScal * pScal; // D uses same scalars but prop-aware ratio
    const ffMult = 1 + (vScal - 1) * 0.5; // FF — voltage-only half correction (prop size doesn't shift FF)

    const rollBase = base;
    const pitchBase = base + 3;
    const yawBase = base * 0.945;

    const ff = FF_BY_STYLE[style] || FF_BY_STYLE.Bando;

    return {
        roll_p: clamp(Math.round(rollBase * fullMult), 20, 90),
        roll_i: Math.round(rollBase * 1.902 * halfMult),
        roll_d: Math.round(rollBase * dr * dMult),
        dMax_roll: Math.round(rollBase * dr * dMult),
        roll_f: Math.round(ff.roll_f * ffMult),
        pitch_p: clamp(Math.round(pitchBase * fullMult), 20, 90),
        pitch_i: Math.round(pitchBase * 1.902 * halfMult),
        pitch_d: Math.round(pitchBase * dr * dMult),
        dMax_pitch: Math.round(pitchBase * dr * dMult),
        pitch_f: Math.round(ff.pitch_f * ffMult),
        yaw_p: clamp(Math.round(yawBase * fullMult), 15, 70),
        yaw_i: Math.round(yawBase * 1.902 * halfMult),
        yaw_d: 0,
        yaw_f: Math.round(ff.yaw_f * ffMult),
        d_min_roll: Math.round(rollBase * dr * 0.887 * dMult),
        d_min_pitch: Math.round(pitchBase * dr * 0.887 * dMult),
    };
}

function filterRecommendation(prop) {
    prop = Number.parseFloat(prop);
    if (prop <= 3) {
        return { hz: 450, low: 400, high: 500, note: "Small / Micro" };
    }
    if (prop <= 4) {
        return { hz: 380, low: 350, high: 420, note: "4-inch" };
    }
    if (prop <= 5.5) {
        return { hz: 300, low: 280, high: 350, note: "5-inch (most common)" };
    }
    if (prop <= 7) {
        return { hz: 250, low: 220, high: 280, note: "6–7 inch" };
    }
    if (prop <= 10) {
        return { hz: 180, low: 150, high: 220, note: "8–10 inch" };
    }
    return { hz: 120, low: 100, high: 150, note: '10"+ Large' };
}

// ─────────────────────────────────────────────────────────────────────────────
// Log Analyzer (ported from V5.4 JS implementation with V5.6 analysis logic)
// ─────────────────────────────────────────────────────────────────────────────

function parseBlackboxCSV(text) {
    const lines = text.split(/\r?\n/);
    let headerIdx = -1;
    for (let i = 0; i < lines.length; i++) {
        if (lines[i].includes("loopIteration")) {
            headerIdx = i;
            break;
        }
    }
    if (headerIdx === -1) {
        return null;
    }

    const headers = lines[headerIdx].split(",").map((h) => h.trim().replaceAll(/^"|"$/g, ""));
    const rows = [];
    for (let i = headerIdx + 1; i < lines.length; i++) {
        const parts = lines[i].split(",");
        if (parts.length < 2) {
            continue;
        }
        const row = {};
        headers.forEach((h, idx) => {
            const raw = (parts[idx] || "").trim();
            const n = Number(raw);
            row[h] = Number.isNaN(n) ? raw : n;
        });
        rows.push(row);
    }
    return rows;
}

// Display labels for tracking ratios (not used in scoring)
function trackingLabel(ratio) {
    if (ratio === null) {
        return "NO DATA";
    }
    if (ratio >= 0.98 && ratio <= 1.02) {
        return "EXCELLENT";
    }
    if (ratio >= 0.92 && ratio <= 1.08) {
        return "GOOD";
    }
    if (ratio >= 0.8 && ratio <= 1.2) {
        return "FAIR";
    }
    return "POOR";
}

// Analyse P-gain quality from step input events.
function analyzePGain(rows, axis, isLevelMode) {
    const spKey = `setpoint[${axis}]`,
        gyroKey = `gyroADC[${axis}]`;
    const overshootPcts = [],
        lagFrames = [],
        zeroCrossingCounts = [];

    for (let i = 1; i + 30 < rows.length; i++) {
        const spPrev = Number(rows[i - 1][spKey] ?? 0);
        const spCurr = Number(rows[i][spKey] ?? 0);
        if (Math.abs(spCurr - spPrev) <= 20) {
            continue;
        }
        const absSP = Math.abs(spCurr);
        if (absSP < 5) {
            continue;
        }

        let peakGyro = Math.abs(Number(rows[i][gyroKey] ?? 0)),
            peakFrame = i;
        for (let j = 1; j <= 30 && i + j < rows.length; j++) {
            const g = Math.abs(Number(rows[i + j][gyroKey] ?? 0));
            if (g > peakGyro) {
                peakGyro = g;
                peakFrame = i + j;
            }
        }
        overshootPcts.push(((peakGyro - absSP) / absSP) * 100);

        let lagFound = false;
        for (let j = 1; j <= 30 && i + j < rows.length; j++) {
            if (Math.abs(Number(rows[i + j][gyroKey] ?? 0)) >= absSP * 0.9) {
                lagFrames.push(j);
                lagFound = true;
                break;
            }
        }
        if (!lagFound) {
            lagFrames.push(30);
        }

        if (isLevelMode) {
            let crossings = 0;
            for (let j = 1; j <= 20 && peakFrame + j < rows.length; j++) {
                const e1 = Number(rows[peakFrame + j - 1][gyroKey] ?? 0) - Number(rows[peakFrame + j - 1][spKey] ?? 0);
                const e2 = Number(rows[peakFrame + j][gyroKey] ?? 0) - Number(rows[peakFrame + j][spKey] ?? 0);
                if (e1 * e2 < 0) {
                    crossings++;
                }
            }
            zeroCrossingCounts.push(crossings);
        }
    }
    return { overshootPcts, lagFrames, zeroCrossingCounts };
}

// Analyse D-gain quality from step input events and high-throttle noise.
function analyzeDGain(rows, axis) {
    const spKey = `setpoint[${axis}]`,
        gyroKey = `gyroADC[${axis}]`;
    const gyroUnfKey = `gyroUnfilt[${axis}]`,
        axisDKey = `axisD[${axis}]`,
        axisPKey = `axisP[${axis}]`;
    const zeroCrossingCounts = [],
        dToPRatios = [];
    let hiThrDOscCount = 0,
        hiThrCount = 0;

    for (let i = 1; i + 21 < rows.length; i++) {
        const spPrev = Number(rows[i - 1][spKey] ?? 0);
        const spCurr = Number(rows[i][spKey] ?? 0);
        if (Math.abs(spCurr - spPrev) <= 20 || Math.abs(spCurr) < 5) {
            continue;
        }

        let peakFrame = i,
            peakGyro = Math.abs(Number(rows[i][gyroKey] ?? 0));
        for (let j = 1; j <= 30 && i + j < rows.length; j++) {
            const g = Math.abs(Number(rows[i + j][gyroKey] ?? 0));
            if (g > peakGyro) {
                peakGyro = g;
                peakFrame = i + j;
            }
        }

        let crossings = 0;
        for (let j = 1; j <= 20 && peakFrame + j < rows.length; j++) {
            const e1 = Number(rows[peakFrame + j - 1][gyroKey] ?? 0) - Number(rows[peakFrame + j - 1][spKey] ?? 0);
            const e2 = Number(rows[peakFrame + j][gyroKey] ?? 0) - Number(rows[peakFrame + j][spKey] ?? 0);
            if (e1 * e2 < 0) {
                crossings++;
            }
        }
        zeroCrossingCounts.push(crossings);

        const dVal = Math.abs(Number(rows[peakFrame][axisDKey] ?? 0));
        const pVal = Math.abs(Number(rows[peakFrame][axisPKey] ?? 0));
        if (pVal > 1) {
            dToPRatios.push(dVal / pVal);
        }
    }

    for (const row of rows) {
        if (Number(row["rcCommand[3]"] ?? 1000) > 1400) {
            hiThrCount++;
            const unfilt = Math.abs(Number(row[gyroUnfKey] ?? 0));
            const filt = Math.abs(Number(row[gyroKey] ?? 0));
            if (unfilt > filt * 2 && Math.abs(Number(row[axisDKey] ?? 0)) > 20) {
                hiThrDOscCount++;
            }
        }
    }

    return {
        avgCrossings:
            zeroCrossingCounts.length > 0
                ? zeroCrossingCounts.reduce((a, b) => a + b, 0) / zeroCrossingCounts.length
                : 0,
        avgDtoP: dToPRatios.length > 0 ? dToPRatios.reduce((a, b) => a + b, 0) / dToPRatios.length : 0,
        filterNoiseDOsc: hiThrCount > 0 && hiThrDOscCount / hiThrCount > 0.3,
        stepCount: zeroCrossingCounts.length,
    };
}

function analyzeLog(rows, motorTemp = "WARM", config = null) {
    if (!rows || rows.length === 0) {
        return { error: "No valid data found in log." };
    }

    const totalFrames = rows.length;

    // ── RPM FILTER DETECTION ──────────────────────────────────────────────────
    // RPM filter is ACTIVE when dshot_bidir == 1 AND gyro_rpm_notch_harmonics > 0.
    // Config header fields: rpmFilter.harmonics (from gyro_rpm_notch_harmonics),
    // motor.dshotBidir (from dshot_bidir), motor.poles (from motor_poles).
    // Also check raw header as fallback for field name variations.
    const rawHeader = config?._raw ?? {};
    const rpmHarmonics =
        config?.rpmFilter?.harmonics ??
        parseInt(rawHeader["gyro_rpm_notch_harmonics"] ?? rawHeader["rpm_filter_harmonics"] ?? "0", 10);
    const motorPoles = config?.motor?.poles ?? parseInt(rawHeader["motor_poles"] ?? "14", 10);
    const dshotBidir = config?.motor?.dshotBidir ?? parseInt(rawHeader["dshot_bidir"] ?? "0", 10);
    const hasEpmFields = Object.keys(rows[0]).some((k) => /erpm/i.test(k) || /rpm\[/i.test(k));
    const rpmFilterActive = (dshotBidir === 1 && rpmHarmonics > 0) || (hasEpmFields && rpmHarmonics > 0);

    // ── THROTTLE ZONE SPLIT ───────────────────────────────────────────────────
    // Low 1000-1570: P/D tracking analysis.  High 1570-2000: filter/noise analysis.
    const lowZoneRows = [];
    const highZoneRows = [];
    for (const row of rows) {
        const thr = Number(row["rcCommand[3]"] ?? 1000);
        if (thr <= 1570) {
            lowZoneRows.push(row);
        } else {
            highZoneRows.push(row);
        }
    }
    const highZonePct = (highZoneRows.length / totalFrames) * 100;
    const lowHighThrottleWarning = highZonePct < 5;

    // ── LOW THROTTLE ZONE: TRACKING ANALYSIS ──────────────────────────────────
    // Setpoint vs gyro tracking, zero-crossing, P oscillation — scoped to low zone.
    const rollTrackingArr = [],
        pitchTrackingArr = [];
    const activeGyroRoll = [];

    for (const row of lowZoneRows) {
        const spRoll = Math.abs(Number(row["setpoint[0]"] ?? 0));
        const spPitch = Math.abs(Number(row["setpoint[1]"] ?? 0));
        const gyroRoll = Number(row["gyroADC[0]"] ?? 0);
        const gyroPitch = Number(row["gyroADC[1]"] ?? 0);
        if (spRoll > 50) {
            rollTrackingArr.push(Math.abs(gyroRoll) / spRoll);
            activeGyroRoll.push(gyroRoll);
        }
        if (spPitch > 50) {
            pitchTrackingArr.push(Math.abs(gyroPitch) / spPitch);
        }
    }

    const rollTrackingRatio =
        rollTrackingArr.length > 0 ? rollTrackingArr.reduce((a, b) => a + b, 0) / rollTrackingArr.length : null;
    const pitchTrackingRatio =
        pitchTrackingArr.length > 0 ? pitchTrackingArr.reduce((a, b) => a + b, 0) / pitchTrackingArr.length : null;

    const rollTracking = { label: trackingLabel(rollTrackingRatio) };
    const pitchTracking = { label: trackingLabel(pitchTrackingRatio) };

    // Zero crossing rate in low zone
    let zeroCrossings = 0;
    for (let j = 1; j < activeGyroRoll.length; j++) {
        if (activeGyroRoll[j - 1] >= 0 !== activeGyroRoll[j] >= 0) {
            zeroCrossings++;
        }
    }
    const zeroCrossingRate = activeGyroRoll.length > 0 ? (zeroCrossings / activeGyroRoll.length) * 100 : 0;
    let zcLabel;
    if (zeroCrossingRate < 1) {
        zcLabel = "EXCELLENT";
    } else if (zeroCrossingRate < 3) {
        zcLabel = "GOOD";
    } else if (zeroCrossingRate < 10) {
        zcLabel = "FAIR";
    } else {
        zcLabel = "POOR";
    }

    // Propwash detection in low zone
    let propwashDetected = false;
    for (let i = 1; i < lowZoneRows.length && !propwashDetected; i++) {
        const tPrev = Number(lowZoneRows[i - 1]["rcCommand[3]"] ?? 1000);
        const tCurr = Number(lowZoneRows[i]["rcCommand[3]"] ?? 1000);
        if (tPrev - tCurr > 200) {
            let osc = 0;
            for (let j = 1; j <= 30 && i + j < lowZoneRows.length; j++) {
                const g1 = Number(lowZoneRows[i + j - 1]["gyroADC[0]"] ?? 0);
                const g2 = Number(lowZoneRows[i + j]["gyroADC[0]"] ?? 0);
                if (g1 * g2 < 0) {
                    osc++;
                }
            }
            if (osc >= 3) {
                propwashDetected = true;
            }
        }
    }

    // ── HIGH THROTTLE ZONE: NOISE / FILTER ANALYSIS ───────────────────────────

    // Estimate sample rate from config looptime or assume 3.2kHz
    const looptimeUs = config?.misc?.looptime ?? 312;
    const sampleRate = 1e6 / looptimeUs;

    // FFT noise peak detection on gyroADC in high throttle zone
    const fftNoisePeaks = [];
    let spectrogramData = null;
    if (highZoneRows.length >= 64) {
        // Collect gyroADC[0] (roll axis — representative) for high-throttle FFT
        const gyroSignal = highZoneRows.map((r) => Number(r["gyroADC[0]"] ?? 0));
        const fftSize = _nextPow2(Math.min(gyroSignal.length, 4096));
        const hann = _hannWindow(fftSize);
        const re = new Float64Array(fftSize);
        const im = new Float64Array(fftSize);
        for (let i = 0; i < fftSize; i++) {
            re[i] = (gyroSignal[i] ?? 0) * hann[i];
            im[i] = 0;
        }
        fftInPlace(re, im);

        // Compute magnitude spectrum (only positive frequencies)
        const halfN = fftSize >> 1;
        const freqBinHz = sampleRate / fftSize;
        const mag = new Float64Array(halfN);
        for (let k = 0; k < halfN; k++) {
            mag[k] = Math.sqrt(re[k] * re[k] + im[k] * im[k]) / fftSize;
        }

        // Find top 3 peaks above 80Hz
        const minBin = Math.ceil(80 / freqBinHz);
        const maxBin = Math.min(halfN, Math.floor(500 / freqBinHz));
        const peakCandidates = [];
        for (let k = minBin + 1; k < maxBin - 1; k++) {
            if (mag[k] > mag[k - 1] && mag[k] > mag[k + 1]) {
                peakCandidates.push({ freq: k * freqBinHz, amplitude: mag[k], bin: k });
            }
        }
        peakCandidates.sort((a, b) => b.amplitude - a.amplitude);
        const maxAmp = peakCandidates.length > 0 ? peakCandidates[0].amplitude : 1;

        // Dynamic notch range
        const dynNotchMin = config?.dynamicNotch?.minHz ?? 100;
        const dynNotchMax = config?.dynamicNotch?.maxHz ?? 600;
        const dynNotchCount = config?.dynamicNotch?.count ?? 0;

        for (let p = 0; p < Math.min(3, peakCandidates.length); p++) {
            const pk = peakCandidates[p];
            const relAmp = ((pk.amplitude / maxAmp) * 100).toFixed(0);
            // Check coverage: RPM filter covers motor harmonics, dynamic notch covers its range
            let coverage = "UNCOVERED";
            let coverageIcon = "⚠️";
            if (rpmFilterActive) {
                // RPM filter covers motor fundamental and harmonics — check if peak is near one
                const motorFreqs = _estimateMotorFreqs(highZoneRows, motorPoles);
                if (motorFreqs.fundamental > 0) {
                    for (let h = 1; h <= rpmHarmonics; h++) {
                        if (Math.abs(pk.freq - motorFreqs.fundamental * h) < motorFreqs.fundamental * 0.15) {
                            coverage = "RPM filter";
                            coverageIcon = "✅";
                            break;
                        }
                    }
                }
            }
            if (coverage === "UNCOVERED" && dynNotchCount > 0 && pk.freq >= dynNotchMin && pk.freq <= dynNotchMax) {
                coverage = "dynamic notch tracking";
                coverageIcon = "✅";
            }
            fftNoisePeaks.push({
                freq: Math.round(pk.freq),
                relAmplitude: relAmp,
                coverage,
                coverageIcon,
            });
        }

        // ── MINI SPECTROGRAM DATA ─────────────────────────────────────────────
        // Build spectrogram from ALL frames (not just high throttle) for full-flight view
        spectrogramData = _buildSpectrogramData(rows, sampleRate);
    }

    // ── RPM HARMONIC ALIGNMENT ────────────────────────────────────────────────
    let rpmAlignment = null;
    if (rpmFilterActive && highZoneRows.length >= 64) {
        const motorFreqs = _estimateMotorFreqs(highZoneRows, motorPoles);
        if (motorFreqs.fundamental > 0) {
            const alignResults = [];
            for (let h = 1; h <= 3; h++) {
                const harmFreq = motorFreqs.fundamental * h;
                if (harmFreq > 500) break;
                // Check if this harmonic shows up as an uncovered peak
                const leaked = fftNoisePeaks.some(
                    (pk) => Math.abs(pk.freq - harmFreq) < motorFreqs.fundamental * 0.15 && pk.coverage === "UNCOVERED",
                );
                alignResults.push({
                    harmonic: h,
                    freq: Math.round(harmFreq),
                    aligned: !leaked,
                });
            }
            rpmAlignment = {
                fundamental: Math.round(motorFreqs.fundamental),
                harmonics: alignResults,
            };
        }
    }

    // ── D-TERM NOISE ASSESSMENT ───────────────────────────────────────────────
    let dTermNoise = { lowZone: "N/A", highZone: "N/A", lowRms: 0, highRms: 0 };
    const dTermLowVals = [];
    const dTermHighVals = [];
    for (const row of lowZoneRows) {
        const d0 = Number(row["axisD[0]"] ?? 0);
        const d1 = Number(row["axisD[1]"] ?? 0);
        dTermLowVals.push(d0 * d0 + d1 * d1);
    }
    for (const row of highZoneRows) {
        const d0 = Number(row["axisD[0]"] ?? 0);
        const d1 = Number(row["axisD[1]"] ?? 0);
        dTermHighVals.push(d0 * d0 + d1 * d1);
    }
    const lowDRms =
        dTermLowVals.length > 0 ? Math.sqrt(dTermLowVals.reduce((a, b) => a + b, 0) / dTermLowVals.length) : 0;
    const highDRms =
        dTermHighVals.length > 0 ? Math.sqrt(dTermHighVals.reduce((a, b) => a + b, 0) / dTermHighVals.length) : 0;
    dTermNoise = {
        lowZone: _dTermNoiseLabel(lowDRms),
        highZone: _dTermNoiseLabel(highDRms),
        lowRms: lowDRms.toFixed(1),
        highRms: highDRms.toFixed(1),
    };

    // ── P GAIN ANALYSIS (scoped to low throttle zone) ─────────────────────────
    const ANGLE_MODE_FLAG = 2;
    let levelModeFrames = 0;
    for (const row of rows) {
        if (Number(row["flightModeFlags"] ?? 0) & ANGLE_MODE_FLAG) {
            levelModeFrames++;
        }
    }
    const isLevelMode = rows.length > 0 && levelModeFrames / rows.length > 0.5;

    const rollPData = analyzePGain(lowZoneRows, 0, isLevelMode);
    const pitchPData = analyzePGain(lowZoneRows, 1, isLevelMode);

    const allOvershoots = [...rollPData.overshootPcts, ...pitchPData.overshootPcts];
    const allLags = [...rollPData.lagFrames, ...pitchPData.lagFrames];
    const allCrossings = [...rollPData.zeroCrossingCounts, ...pitchPData.zeroCrossingCounts];

    let pVerdict, pAction;
    if (allOvershoots.length === 0) {
        pVerdict = "NO STEP INPUTS DETECTED";
        pAction = "No rapid stick inputs found. Fly with sharp, deliberate inputs to enable P analysis.";
    } else if (isLevelMode) {
        const modeNote =
            "Level mode detected — single overshoots ignored (auto-leveler), watching for oscillations only\n";
        const avgOsc = allCrossings.length > 0 ? allCrossings.reduce((a, b) => a + b, 0) / allCrossings.length : 0;
        const avgOvershoot = allOvershoots.reduce((a, b) => a + b, 0) / allOvershoots.length;
        const lowRespCount = allOvershoots.filter((o) => o < -70).length;
        const lowRespRatio = lowRespCount / allOvershoots.length;
        if (avgOsc >= 3) {
            pVerdict = "P TOO HIGH ⚠";
            pAction = `${modeNote}Average oscillations after step: ${avgOsc.toFixed(1)} zero-crossings.\nP too high — reduce by 5–10. Symptom: bounce-back after flips/rolls.`;
        } else if (lowRespRatio > 0.5) {
            pVerdict = "P TOO LOW";
            pAction = `${modeNote}Gyro response below 30% of setpoint in ${Math.round(lowRespRatio * 100)}% of step inputs.\nP too low — increase by 5–10. Symptom: slow/sloppy response.`;
        } else {
            pVerdict = "P GAINS LOOK GOOD ✓";
            pAction = `${modeNote}Average oscillations: ${avgOsc.toFixed(1)}, overshoot: ${avgOvershoot.toFixed(1)}%. P tracking well in level mode.`;
        }
    } else {
        const avgOvershoot = allOvershoots.reduce((a, b) => a + b, 0) / allOvershoots.length;
        const avgLag = allLags.reduce((a, b) => a + b, 0) / allLags.length;
        if (avgOvershoot > 15) {
            pVerdict = "P TOO HIGH ⚠";
            pAction = `Average overshoot: ${avgOvershoot.toFixed(1)}%.\nP too high — reduce by 5–10. Symptom: bounce-back after flips/rolls.`;
        } else if (avgLag > 3 && avgOvershoot < 5) {
            pVerdict = "P TOO LOW";
            pAction = `Average response lag: ${avgLag.toFixed(1)} frames.\nP too low — increase by 5–10. Symptom: slow/sloppy response.`;
        } else {
            pVerdict = "P GAINS LOOK GOOD ✓";
            pAction = `Average overshoot: ${avgOvershoot.toFixed(1)}%, lag: ${avgLag.toFixed(1)} frames. P tracking well.`;
        }
    }

    if (pVerdict === "P TOO HIGH ⚠") {
        const avgTracking = ((rollTrackingRatio ?? 1) + (pitchTrackingRatio ?? 1)) / 2;
        if (zeroCrossingRate <= 5 || avgTracking <= 1.15) {
            pVerdict = "P LOOKS ACCEPTABLE";
            pAction =
                "Overshoot pattern detected in step inputs, but zero-crossing rate and tracking ratio do not both confirm P is too high.\nMonitor during flight — no P reduction recommended based on available evidence.";
        }
    }

    // ── D GAIN ANALYSIS (scoped to low throttle zone) ─────────────────────────
    const rollDData = analyzeDGain(lowZoneRows, 0);
    const pitchDData = analyzeDGain(lowZoneRows, 1);
    const avgDCrossings = (rollDData.avgCrossings + pitchDData.avgCrossings) / 2;
    const avgDtoP = (rollDData.avgDtoP + pitchDData.avgDtoP) / 2;
    const filterNoiseDOsc = rollDData.filterNoiseDOsc || pitchDData.filterNoiseDOsc;
    const hasSteps = rollDData.stepCount > 0 || pitchDData.stepCount > 0;

    let dVerdict, dAction;
    if (!hasSteps) {
        dVerdict = "NO STEP INPUTS DETECTED";
        dAction = "No rapid stick inputs found. Fly with sharp, deliberate inputs to enable D analysis.";
    } else if (filterNoiseDOsc) {
        dVerdict = "FILTER NOISE LIMITING D ⚠";
        dAction =
            "Unfiltered gyro is much noisier than filtered at high throttle, and D is still active.\nFix filters before increasing D — see FILTERS section.";
    } else if (avgDCrossings > 3) {
        dVerdict = "D TOO LOW ⚠";
        dAction = `Average zero-crossings after peak: ${avgDCrossings.toFixed(1)}.\nD too low — increase by 3–5. Symptom: propwash oscillations after throttle cuts.`;
    } else if (avgDtoP > 1.5 && motorTemp === "HOT") {
        dVerdict = "D TOO HIGH ⚠";
        dAction = `D/P ratio is ${avgDtoP.toFixed(2)} and motors are running HOT.\nD too high — reduce by 3–5. Check motor temps after flying.`;
    } else if (avgDtoP > 1.5) {
        if (propwashDetected) {
            dVerdict = "D MAY BE TOO HIGH ⚠";
            dAction = `D/P ratio is ${avgDtoP.toFixed(2)} and propwash was detected.\nCheck motor temps and consider reducing D by 3–5.`;
        } else {
            dVerdict = "D WITHIN RANGE";
            dAction = `D/P ratio is ${avgDtoP.toFixed(2)}. Without confirmed propwash or HOT motors, no D reduction recommended at this time.`;
        }
    } else {
        dVerdict = "D GAINS LOOK GOOD ✓";
        dAction = `Average zero-crossings: ${avgDCrossings.toFixed(1)}, D/P ratio: ${avgDtoP.toFixed(2)}. D gains look good.`;
    }

    if (motorTemp === "HOT" && dVerdict === "D GAINS LOOK GOOD ✓") {
        dAction += "\nD gain too high or insufficient filtering — reduce D by 5–10 or increase filtering (motors HOT).";
    } else if (motorTemp === "COOL") {
        dAction += "\nD gain may have headroom — could increase slightly (motors COOL after flight).";
    }
    if (propwashDetected) {
        dAction += "\nPropwash detected — increase D by 3–5 or check filtering.";
    }

    // ── D_MAX FLIGHT 2 REFINEMENT ─────────────────────────────────────────────
    const avgOvershootAll =
        allOvershoots.length > 0 ? allOvershoots.reduce((a, b) => a + b, 0) / allOvershoots.length : null;

    let dMaxRefinement = null;
    if (config && avgOvershootAll !== null && avgOvershootAll >= 25 && avgOvershootAll <= 35) {
        const dMaxRoll = config.pids?.roll?.[3] ?? null;
        const dMaxPitch = config.pids?.pitch?.[3] ?? null;
        const dMaxAdvance = config.pids?.dMaxAdvance ?? null;
        if (dMaxRoll !== null && dMaxPitch !== null && Math.abs(dMaxRoll - 40) <= 3 && Math.abs(dMaxPitch - 46) <= 3) {
            dMaxRefinement = {
                dMaxRoll,
                dMaxPitch,
                dMaxAdvance: dMaxAdvance ?? 20,
                suggestRoll: 35,
                suggestPitch: 38,
                suggestAdvance: 10,
                avgOvershoot: avgOvershootAll,
            };
        }
    }

    // ── FILTER SUGGESTIONS (only when genuinely wrong) ────────────────────────
    const filterSuggestions = [];
    if (!rpmFilterActive) {
        filterSuggestions.push("Enable RPM filter — most effective filter available, requires bidirectional DShot.");
    }
    // Only suggest LP1/LP2 changes when RPM filter is NOT active
    if (!rpmFilterActive) {
        const lpf2Hz = config?.gyroFilters?.lowpass2Hz;
        if (highDRms > 40 && lpf2Hz !== null && lpf2Hz !== undefined && lpf2Hz > 0 && lpf2Hz < 500) {
            const reduction = highDRms > 80 ? 100 : highDRms > 60 ? 50 : 30;
            const suggested = Math.max(80, lpf2Hz - reduction);
            filterSuggestions.push(`set gyro_lpf2_static_hz = ${suggested}  # was ${lpf2Hz}`);
        }
    }
    // Dynamic notch range suggestions (valid with or without RPM filter)
    for (const pk of fftNoisePeaks) {
        if (pk.coverage === "UNCOVERED") {
            const dynMin = config?.dynamicNotch?.minHz ?? 100;
            const dynMax = config?.dynamicNotch?.maxHz ?? 600;
            if (pk.freq < dynMin) {
                filterSuggestions.push(
                    `set dyn_notch_min_hz = ${Math.max(50, pk.freq - 20)}  # was ${dynMin} — uncovered peak at ${pk.freq}Hz`,
                );
            } else if (pk.freq > dynMax) {
                filterSuggestions.push(
                    `set dyn_notch_max_hz = ${pk.freq + 30}  # was ${dynMax} — uncovered peak at ${pk.freq}Hz`,
                );
            }
            break; // Only suggest for the strongest uncovered peak
        }
    }

    return {
        totalFrames,
        lowZoneFrames: lowZoneRows.length,
        highZoneFrames: highZoneRows.length,
        highZonePct: highZonePct.toFixed(1),
        lowHighThrottleWarning,
        rpmFilterActive,
        rpmHarmonics,
        rollTrackingRatio,
        pitchTrackingRatio,
        rollTracking,
        pitchTracking,
        zeroCrossingRate: zeroCrossingRate.toFixed(2),
        zcLabel,
        propwashDetected,
        fftNoisePeaks,
        rpmAlignment,
        dTermNoise,
        pVerdict,
        pAction,
        dVerdict,
        dAction,
        motorTemp,
        config,
        dMaxRefinement,
        filterSuggestions,
        spectrogramData,
    };
}

// ── Helper: estimate motor fundamental frequency from eRPM fields ─────────
function _estimateMotorFreqs(highZoneRows, motorPoles) {
    const erpmKeys = Object.keys(highZoneRows[0] || {}).filter((k) => /erpm/i.test(k) || /motor\[/i.test(k));
    if (erpmKeys.length === 0) return { fundamental: 0 };

    let erpmSum = 0,
        erpmCount = 0;
    for (const row of highZoneRows) {
        for (const key of erpmKeys) {
            const val = Math.abs(Number(row[key] ?? 0));
            if (val > 100) {
                erpmSum += val;
                erpmCount++;
            }
        }
    }
    if (erpmCount === 0) return { fundamental: 0 };
    const avgErpm = erpmSum / erpmCount;
    // fundamental = eRPM / 60 / (motor_poles / 2)
    const polePairs = Math.max(1, motorPoles / 2);
    const fundamental = avgErpm / 60 / polePairs;
    return { fundamental };
}

// ── Helper: D-term noise label from RMS value ─────────────────────────────
function _dTermNoiseLabel(rms) {
    if (rms < 15) return "LOW";
    if (rms < 40) return "MODERATE";
    return "HIGH";
}

// ── Helper: build spectrogram data for canvas rendering ───────────────────
function _buildSpectrogramData(rows, sampleRate) {
    const signal = rows.map((r) => Number(r["gyroADC[0]"] ?? 0));
    const windowSize = 256;
    const hopSize = 128;
    const maxFreqHz = 500;
    const hann = _hannWindow(windowSize);
    const fftN = _nextPow2(windowSize);
    const halfN = fftN >> 1;
    const freqBinHz = sampleRate / fftN;
    const maxBin = Math.min(halfN, Math.ceil(maxFreqHz / freqBinHz));

    const slices = [];
    for (let start = 0; start + windowSize <= signal.length; start += hopSize) {
        const re = new Float64Array(fftN);
        const im = new Float64Array(fftN);
        for (let i = 0; i < windowSize; i++) {
            re[i] = signal[start + i] * hann[i];
        }
        fftInPlace(re, im);
        const slice = new Float64Array(maxBin);
        for (let k = 0; k < maxBin; k++) {
            slice[k] = Math.sqrt(re[k] * re[k] + im[k] * im[k]) / fftN;
        }
        slices.push(slice);
    }

    return {
        slices,
        freqBinHz,
        maxBin,
        maxFreqHz,
        sampleRate,
    };
}

// ─────────────────────────────────────────────────────────────────────────────
// Freq-vs-Throttle Spectrogram (2D heatmap) — adapted from Blackbox Explorer
// ─────────────────────────────────────────────────────────────────────────────
function _buildFreqVsThrottleData(rows, sampleRate) {
    const NUM_THROTTLE_BINS = 100;
    const CHUNK_MS = 300;
    const chunkLen = Math.max(64, Math.round((sampleRate * CHUNK_MS) / 1000));
    const fftSize = _nextPow2(chunkLen);
    const halfN = fftSize >> 1;
    const freqBinHz = sampleRate / fftSize;
    const maxFreqHz = 500;
    const maxBin = Math.min(halfN, Math.ceil(maxFreqHz / freqBinHz));
    const hann = _hannWindow(fftSize);
    const hopLen = Math.max(1, Math.floor(chunkLen / 2));

    // Matrix: [throttleBin][freqBin] accumulator
    const matrix = [];
    const counts = new Int32Array(NUM_THROTTLE_BINS);
    for (let i = 0; i < NUM_THROTTLE_BINS; i++) {
        matrix.push(new Float64Array(maxBin));
    }

    const signal = rows.map((r) => Number(r["gyroADC[0]"] ?? 0));
    const throttles = rows.map((r) => Number(r["rcCommand[3]"] ?? 1000));

    for (let start = 0; start + chunkLen <= signal.length; start += hopLen) {
        // Average throttle for this chunk
        let thrSum = 0;
        for (let i = start; i < start + chunkLen; i++) thrSum += throttles[i];
        const avgThr = thrSum / chunkLen;
        const thrPct = Math.max(0, Math.min(99.9, (avgThr - 1000) / 10));
        const thrBin = Math.floor(thrPct);

        // FFT this chunk
        const re = new Float64Array(fftSize);
        const im = new Float64Array(fftSize);
        for (let i = 0; i < chunkLen && i < fftSize; i++) {
            re[i] = signal[start + i] * hann[i];
        }
        fftInPlace(re, im);

        // Accumulate magnitudes
        for (let k = 0; k < maxBin; k++) {
            matrix[thrBin][k] += Math.sqrt(re[k] * re[k] + im[k] * im[k]) / fftSize;
        }
        counts[thrBin]++;
    }

    // Average each bin
    for (let t = 0; t < NUM_THROTTLE_BINS; t++) {
        if (counts[t] > 1) {
            for (let k = 0; k < maxBin; k++) {
                matrix[t][k] /= counts[t];
            }
        }
    }

    return { matrix, counts, maxBin, freqBinHz, maxFreqHz, sampleRate };
}

// ─────────────────────────────────────────────────────────────────────────────
// Extended Analysis — ported from aerotune7
// ─────────────────────────────────────────────────────────────────────────────

function _analyzeThrottleBands(rows) {
    const BAND_COUNT = 10;
    const sumSq = new Float64Array(BAND_COUNT);
    const count = new Int32Array(BAND_COUNT);

    for (const row of rows) {
        const thr = Number(row["rcCommand[3]"] ?? 1000);
        const thrPct = Math.max(0, Math.min(99.9, (thr - 1000) / 10));
        const band = Math.floor(thrPct / 10);
        // Prefer unfiltered gyro; fall back to filtered
        const noise =
            row["gyroUnfilt[0]"] !== undefined ? Number(row["gyroUnfilt[0]"] ?? 0) : Number(row["gyroADC[0]"] ?? 0);
        sumSq[band] += noise * noise;
        count[band]++;
    }

    const bands = [];
    let maxRms = 0;
    for (let i = 0; i < BAND_COUNT; i++) {
        const rms = count[i] >= 10 ? Math.sqrt(sumSq[i] / count[i]) : null;
        if (rms !== null && rms > maxRms) maxRms = rms;
        bands.push({
            label: `${i * 10}–${(i + 1) * 10}%`,
            rms,
            count: count[i],
        });
    }
    // Normalize for bar display
    for (const b of bands) {
        b.pct = b.rms !== null && maxRms > 0 ? (b.rms / maxRms) * 100 : 0;
    }
    return bands;
}

function _analyzeMotorSpread(rows, motorPoles) {
    const erpmKeys = Object.keys(rows[0] || {}).filter((k) => /erpm/i.test(k) || /motor\[/i.test(k));
    if (erpmKeys.length === 0) return "No eRPM/motor data in log — enable bidirectional DShot for motor telemetry.";

    const BAND_COUNT = 10;

    // Per motor, per throttle band: collect frequencies
    const motorBands = {};
    for (const key of erpmKeys) motorBands[key] = Array.from({ length: BAND_COUNT }, () => []);

    for (const row of rows) {
        const thr = Number(row["rcCommand[3]"] ?? 1000);
        const thrPct = Math.max(0, Math.min(99.9, (thr - 1000) / 10));
        const band = Math.floor(thrPct / 10);
        for (const key of erpmKeys) {
            const val = Math.abs(Number(row[key] ?? 0));
            if (val > 0) {
                // BBL stores eRPM/100; convert to Hz
                const freqHz = (val * 100) / 60;
                motorBands[key][band].push(freqHz);
            }
        }
    }

    const lines = [];
    for (let band = 0; band < BAND_COUNT; band++) {
        const motors = erpmKeys
            .map((key) => {
                const vals = motorBands[key][band];
                if (vals.length < 5) return null;
                vals.sort((a, b) => a - b);
                return {
                    name: key.replace(/[[\]]/g, ""),
                    avg: Math.round(vals.reduce((a, b) => a + b, 0) / vals.length),
                    min: Math.round(vals[0]),
                    max: Math.round(vals[vals.length - 1]),
                    p25: Math.round(vals[Math.floor(vals.length * 0.25)]),
                    p75: Math.round(vals[Math.floor(vals.length * 0.75)]),
                };
            })
            .filter(Boolean);

        if (motors.length === 0) continue;
        const avgs = motors.map((m) => m.avg);
        const spread = Math.max(...avgs) - Math.min(...avgs);
        const spreadLabel = spread > 30 ? "⚠️ HIGH" : spread > 15 ? "MODERATE" : "✅ LOW";
        lines.push(
            `${band * 10}–${(band + 1) * 10}% throttle: spread ${spread}Hz (${spreadLabel})  [${motors.map((m) => `${m.name}:${m.avg}Hz`).join(", ")}]`,
        );
    }
    return lines.length > 0 ? lines.join("\n") : "Insufficient motor data across throttle bands.";
}

function _analyzeStepResponse(rows, sampleRate) {
    const axes = [
        { name: "Roll", spKey: "setpoint[0]", gyroKey: "gyroADC[0]" },
        { name: "Pitch", spKey: "setpoint[1]", gyroKey: "gyroADC[1]" },
    ];
    const samplePeriodMs = 1000 / sampleRate;
    const detFrames = Math.max(5, Math.round(sampleRate * 0.006));
    const afterFrames = Math.round(sampleRate * 0.15);
    const cooldown = Math.round(sampleRate * 0.15);

    const results = [];
    for (const axis of axes) {
        const steps = [];
        let lastStepEnd = 0;
        const iStart = 20 + detFrames;

        for (let i = iStart; i + afterFrames < rows.length; i++) {
            if (i < lastStepEnd + cooldown) continue;
            const spCurr = Number(rows[i][axis.spKey] ?? 0);
            const spPrev = Number(rows[i - detFrames][axis.spKey] ?? 0);
            const delta = spCurr - spPrev;
            if (Math.abs(delta) < 60) continue;

            const direction = delta > 0 ? 1 : -1;
            const baseGyro = Number(rows[i][axis.gyroKey] ?? 0);
            let peakOvershoot = 0;
            let delayFrames = -1;
            let settlingFrames = -1;

            for (let j = 0; j < afterFrames && i + j < rows.length; j++) {
                const g = Number(rows[i + j][axis.gyroKey] ?? 0);
                const overshoot = (g - baseGyro - delta) * direction;
                if (delayFrames < 0 && (g - baseGyro) * direction > Math.abs(delta) * 0.5) delayFrames = j;
                if (settlingFrames < 0 && (g - baseGyro) * direction > Math.abs(delta) * 0.9) settlingFrames = j;
                if (overshoot > peakOvershoot) peakOvershoot = overshoot;
            }

            if (delayFrames < 0) continue;
            if (settlingFrames < 0) settlingFrames = afterFrames;

            const overshootPct = Math.abs(delta) > 0 ? (peakOvershoot / Math.abs(delta)) * 100 : 0;
            if (overshootPct >= 0 && overshootPct < 200 && delayFrames * samplePeriodMs < 500) {
                steps.push({
                    overshootPct,
                    delayMs: delayFrames * samplePeriodMs,
                    settlingMs: settlingFrames * samplePeriodMs,
                });
            }
            lastStepEnd = i + afterFrames;
        }

        if (steps.length < 3) {
            results.push(
                `${axis.name}: Insufficient step data (${steps.length} clean steps). Fly with sharper stick inputs.`,
            );
            continue;
        }

        const avgOS = steps.reduce((a, s) => a + s.overshootPct, 0) / steps.length;
        const avgDelay = steps.reduce((a, s) => a + s.delayMs, 0) / steps.length;
        const avgSettle = steps.reduce((a, s) => a + s.settlingMs, 0) / steps.length;

        let diagnosis;
        if (avgOS > 25) diagnosis = `Heavy overshoot (${avgOS.toFixed(0)}%) — P too high or D too low`;
        else if (avgOS > 15) diagnosis = `Moderate overshoot (${avgOS.toFixed(0)}%) — could use more D`;
        else if (avgOS < 3) diagnosis = `Very low overshoot (${avgOS.toFixed(0)}%) — could handle more P`;
        else diagnosis = `Tracking looks good (${avgOS.toFixed(0)}% overshoot)`;

        results.push(
            `${axis.name}: ${steps.length} steps | overshoot ${avgOS.toFixed(1)}% | delay ${avgDelay.toFixed(1)}ms | settling ${avgSettle.toFixed(1)}ms\n  → ${diagnosis}`,
        );
    }
    return results.join("\n");
}

function _analyzeItermBias(rows) {
    // Detect I-term wind-up / bias: sustained non-zero I-term average
    const axes = [
        { name: "Roll", key: "axisI[0]" },
        { name: "Pitch", key: "axisI[1]" },
        { name: "Yaw", key: "axisI[2]" },
    ];
    // Check if axisI fields exist
    if (!rows[0] || rows[0]["axisI[0]"] === undefined) return null;

    const lines = [];
    for (const axis of axes) {
        let sum = 0,
            count = 0,
            maxAbs = 0;
        for (const row of rows) {
            const v = Number(row[axis.key] ?? 0);
            sum += v;
            count++;
            if (Math.abs(v) > maxAbs) maxAbs = Math.abs(v);
        }
        if (count === 0) continue;
        const avg = sum / count;
        const biasLabel = Math.abs(avg) > 20 ? "⚠️ SIGNIFICANT BIAS" : Math.abs(avg) > 10 ? "MILD BIAS" : "✅ OK";
        lines.push(`${axis.name}: avg I-term ${avg.toFixed(1)} | peak ±${maxAbs.toFixed(0)} — ${biasLabel}`);
        if (Math.abs(avg) > 20) {
            lines.push(
                `  → ${axis.name} I-term offset suggests ${avg > 0 ? "CG forward/right" : "CG back/left"} bias or motor imbalance.`,
            );
        }
    }
    return lines.length > 0 ? lines.join("\n") : null;
}

function _computeLogPidRecommendations(config, stepText, dTermNoise, motorTemp) {
    if (!config?.pids) return null;

    const oldPids = {};
    const newPids = {};
    for (const ax of ["roll", "pitch", "yaw"]) {
        const arr = config.pids[ax] || [];
        const P = arr[0] ?? 0;
        const I = arr[1] ?? 0;
        const D = arr[2] ?? 0;
        oldPids[ax] = { P, I, D };

        // Parse step response to determine adjustments
        let pAdj = 1.0,
            dAdj = 1.0;
        if (stepText) {
            const axUpper = ax.charAt(0).toUpperCase() + ax.slice(1);
            if (stepText.includes(`${axUpper}:`) && stepText.includes("Heavy overshoot")) {
                pAdj = 0.88;
                dAdj = 1.18;
            } else if (stepText.includes(`${axUpper}:`) && stepText.includes("Moderate overshoot")) {
                dAdj = 1.15;
            } else if (stepText.includes(`${axUpper}:`) && stepText.includes("Very low overshoot")) {
                pAdj = 1.12;
            }
        }

        // D-term noise adjustment
        if (dTermNoise?.highZone === "HIGH" && motorTemp === "HOT") {
            dAdj = Math.min(dAdj, 0.9);
        }

        newPids[ax] = {
            P: Math.round(clamp(P * pAdj, 10, 200)),
            I: I, // Keep I unchanged
            D: ax === "yaw" ? 0 : Math.round(clamp(D * dAdj, 10, 150)),
        };
    }
    return { old: oldPids, new: newPids };
}

// Build motor RPM heatmap data: motors x throttle bands
function _buildMotorHeatmap(rows, motorPoles) {
    const erpmKeys = Object.keys(rows[0] || {}).filter((k) => /erpm/i.test(k) || /motor\[/i.test(k));
    if (erpmKeys.length === 0) return null;

    const BANDS = 10;
    const data = {}; // key -> [band0avg, band1avg, ...]
    const sums = {};
    const counts = {};
    for (const key of erpmKeys) {
        sums[key] = new Float64Array(BANDS);
        counts[key] = new Int32Array(BANDS);
    }

    for (const row of rows) {
        const thr = Number(row["rcCommand[3]"] ?? 1000);
        const band = Math.min(9, Math.floor(Math.max(0, (thr - 1000) / 10) / 10));
        for (const key of erpmKeys) {
            const v = Math.abs(Number(row[key] ?? 0));
            if (v > 0) {
                sums[key][band] += (v * 100) / 60; // eRPM/100 → Hz
                counts[key][band]++;
            }
        }
    }

    for (const key of erpmKeys) {
        data[key] = [];
        for (let b = 0; b < BANDS; b++) {
            data[key].push(counts[key][b] >= 5 ? sums[key][b] / counts[key][b] : null);
        }
    }

    return { motors: erpmKeys, bands: BANDS, data };
}

function formatAnalysisResult(r) {
    if (r.error) {
        return `ERROR: ${r.error}`;
    }
    const SEP = "════════════════════════════════════════════════════";
    const lines = [];

    // ── Frame counts & throttle zone split ────────────────────────────────────
    lines.push(
        `Frames analysed : ${r.totalFrames}`,
        `Low throttle zone  (≤1570): ${r.lowZoneFrames} frames`,
        `High throttle zone (>1570): ${r.highZoneFrames} frames (${r.highZonePct}%)`,
        ``,
    );

    if (r.lowHighThrottleWarning) {
        lines.push(
            `⚠️ Limited high-throttle data (${r.highZonePct}%) — fly with sustained throttle punches for accurate noise analysis`,
            ``,
        );
    }

    // ── RPM FILTER STATUS ─────────────────────────────────────────────────────
    lines.push(SEP, `  RPM FILTER STATUS`, SEP);
    if (r.rpmFilterActive) {
        lines.push(`RPM filter active (${r.rpmHarmonics} harmonics) — gyro lowpass filters not required ✅`);
    } else {
        lines.push(`RPM filter not detected — enable RPM filter + bidirectional DShot for best noise rejection`);
    }
    lines.push(``);

    // ── LOW THROTTLE ZONE: TRACKING QUALITY ───────────────────────────────────
    const rRatio = r.rollTrackingRatio !== null ? r.rollTrackingRatio.toFixed(3) : "N/A";
    const pRatio = r.pitchTrackingRatio !== null ? r.pitchTrackingRatio.toFixed(3) : "N/A";
    lines.push(
        SEP,
        `  LOW THROTTLE ZONE — TRACKING QUALITY`,
        SEP,
        `Roll tracking:  ${rRatio} (target: 1.000) — ${r.rollTracking.label}`,
        `Pitch tracking: ${pRatio} (target: 1.000) — ${r.pitchTracking.label}`,
        `Zero-crossing rate (P oscillation): ${r.zeroCrossingRate}% — ${r.zcLabel}`,
    );
    if (r.propwashDetected) {
        lines.push(`Propwash oscillation detected in throttle cuts`);
    }
    lines.push(``);

    // ── HIGH THROTTLE ZONE: NOISE QUALITY ─────────────────────────────────────
    lines.push(SEP, `  HIGH THROTTLE ZONE — NOISE QUALITY`, SEP);

    // FFT Noise peaks
    if (r.fftNoisePeaks.length > 0) {
        lines.push(`Top noise peaks (gyro FFT, >80Hz):`);
        for (const pk of r.fftNoisePeaks) {
            lines.push(`  Peak at ${pk.freq}Hz (${pk.relAmplitude}% rel) — ${pk.coverage} ${pk.coverageIcon}`);
        }
    } else if (r.highZoneFrames < 64) {
        lines.push(`Insufficient high-throttle data for FFT analysis`);
    } else {
        lines.push(`No significant noise peaks detected above 80Hz ✅`);
    }
    lines.push(``);

    // RPM harmonic alignment
    if (r.rpmAlignment) {
        lines.push(`Motor fundamental at ~${r.rpmAlignment.fundamental}Hz`);
        for (const h of r.rpmAlignment.harmonics) {
            if (h.aligned) {
                lines.push(`  ${h.harmonic}x harmonic (${h.freq}Hz) — RPM filter aligned ✅`);
            } else {
                lines.push(`  ${h.harmonic}x harmonic (${h.freq}Hz) — possible gap in RPM coverage ⚠️`);
            }
        }
        lines.push(``);
    }

    // D-term noise
    lines.push(
        `D-term noise: low zone ${r.dTermNoise.lowZone} (RMS ${r.dTermNoise.lowRms}) | high zone ${r.dTermNoise.highZone} (RMS ${r.dTermNoise.highRms})`,
        ``,
    );

    // ── P GAIN ────────────────────────────────────────────────────────────────
    lines.push(SEP, `  ROLL/PITCH P : ${r.pVerdict}`, SEP);
    lines.push(
        r.pAction,
        `Tip: level mode and acro mode are both valid for tuning. Level mode gives clean repeatable step inputs.`,
        ``,
    );

    // ── D GAIN ────────────────────────────────────────────────────────────────
    lines.push(SEP, `  ROLL/PITCH D : ${r.dVerdict}`, SEP, r.dAction);

    // ── Flight 2 refinement — D_Max headroom ──────────────────────────────────
    if (r.dMaxRefinement) {
        const ref = r.dMaxRefinement;
        lines.push(
            ``,
            SEP,
            `  FLIGHT 2 REFINEMENT — D_MAX HEADROOM`,
            SEP,
            `D_Max is at Betaflight defaults (Roll: ${ref.dMaxRoll}, Pitch: ${ref.dMaxPitch}).`,
            `With ${ref.avgOvershoot.toFixed(1)}% average overshoot the D-term ceiling may be too permissive during fast moves.`,
            ``,
            `Suggested CLI changes:`,
            `  set d_max = ${ref.suggestRoll},${ref.suggestPitch},0  # was ${ref.dMaxRoll},${ref.dMaxPitch},0`,
            `  set d_max_advance = ${ref.suggestAdvance}  # was ${ref.dMaxAdvance}`,
            ``,
            `Re-fly the test pattern and re-analyze. If overshoot drops below 15% these values are correct.`,
        );
    }

    // ── SUGGESTED CHANGES (only when something is genuinely wrong) ────────────
    if (r.filterSuggestions && r.filterSuggestions.length > 0) {
        lines.push(``, SEP, `  SUGGESTED CHANGES`, SEP);
        for (const s of r.filterSuggestions) {
            lines.push(`  ${s}`);
        }
        lines.push(``);
    }

    lines.push(``, `Fresh props recommended before tuning — damaged props create false noise in logs.`);

    return lines.join("\n");
}

// ─────────────────────────────────────────────────────────────────────────────
// SysID: chirp detection, FFT, frequency response, Bode data, PID synthesis
// ─────────────────────────────────────────────────────────────────────────────

/**
 * Radix-2 Cooley-Tukey FFT, in-place.
 * re[] and im[] must both have length equal to a power of 2.
 * On return, re[k]/im[k] contain real/imaginary parts of bin k.
 */
function fftInPlace(re, im) {
    const N = re.length;
    // Bit-reversal permutation
    let j = 0;
    for (let i = 1; i < N; i++) {
        let bit = N >> 1;
        for (; j & bit; bit >>= 1) j ^= bit;
        j ^= bit;
        if (i < j) {
            let t = re[i];
            re[i] = re[j];
            re[j] = t;
            t = im[i];
            im[i] = im[j];
            im[j] = t;
        }
    }
    // Butterfly stages
    for (let len = 2; len <= N; len <<= 1) {
        const half = len >> 1;
        const ang = (-2 * Math.PI) / len;
        const wBaseRe = Math.cos(ang);
        const wBaseIm = Math.sin(ang);
        for (let i = 0; i < N; i += len) {
            let wRe = 1,
                wIm = 0;
            for (let k = 0; k < half; k++) {
                const uRe = re[i + k];
                const uIm = im[i + k];
                const vRe = re[i + k + half] * wRe - im[i + k + half] * wIm;
                const vIm = re[i + k + half] * wIm + im[i + k + half] * wRe;
                re[i + k] = uRe + vRe;
                im[i + k] = uIm + vIm;
                re[i + k + half] = uRe - vRe;
                im[i + k + half] = uIm - vIm;
                const nextWRe = wRe * wBaseRe - wIm * wBaseIm;
                wIm = wRe * wBaseIm + wIm * wBaseRe;
                wRe = nextWRe;
            }
        }
    }
}

function _nextPow2(n) {
    let p = 1;
    while (p < n) p <<= 1;
    return p;
}

function _hannWindow(N) {
    const w = new Float64Array(N);
    for (let i = 0; i < N; i++) w[i] = 0.5 * (1 - Math.cos((2 * Math.PI * i) / (N - 1)));
    return w;
}

/**
 * Unwrap a phase array (radians) in-place.
 */
function _unwrapPhase(phase) {
    for (let i = 1; i < phase.length; i++) {
        let d = phase[i] - phase[i - 1];
        while (d > Math.PI) d -= 2 * Math.PI;
        while (d < -Math.PI) d += 2 * Math.PI;
        phase[i] = phase[i - 1] + d;
    }
}

/**
 * Detect chirp data in a BBL session.
 * Criteria: any parsed row has bit 10 set in flightModeFlags (value 1024).
 */
function detectChirp(parsedRows, config) {
    const hasChirp = parsedRows.some((row) => (parseInt(row.flightModeFlags) & 1024) !== 0);
    return hasChirp;
}

/**
 * Find the longest continuous segment where the std dev of a signal over a
 * rolling windowSize-sample window exceeds `threshold`.
 * Returns { start, end } byte indices.
 */
function _findLongestActiveSegment(signal, windowSize, threshold) {
    const N = signal.length;
    if (N < windowSize) return { start: 0, end: N };

    // Initialise rolling sum / sum-of-squares for first window
    let sum = 0,
        sum2 = 0;
    for (let i = 0; i < windowSize; i++) {
        sum += signal[i];
        sum2 += signal[i] * signal[i];
    }

    let bestStart = 0,
        bestLen = 0;
    let curStart = -1;

    for (let i = windowSize; i <= N; i++) {
        const mean = sum / windowSize;
        const std = Math.sqrt(Math.max(0, sum2 / windowSize - mean * mean));
        const winStart = i - windowSize;

        if (std > threshold) {
            if (curStart < 0) curStart = winStart;
            const curLen = i - curStart;
            if (curLen > bestLen) {
                bestLen = curLen;
                bestStart = curStart;
            }
        } else {
            curStart = -1;
        }

        if (i < N) {
            sum += signal[i];
            sum2 += signal[i] * signal[i];
            sum -= signal[winStart];
            sum2 -= signal[winStart] * signal[winStart];
        }
    }

    if (bestLen < windowSize) return { start: 0, end: Math.min(N, windowSize * 4) };
    return { start: bestStart, end: bestStart + bestLen };
}

/**
 * Welch's method coherence estimate.
 * Returns Float64Array of length segLen/2 + 1 with coherence values in [0, 1].
 * Bin k corresponds to frequency k * sampleRate / segLen.
 */
function _welchCoherence(x, y, segLen) {
    const half = segLen >> 1;
    const win = _hannWindow(segLen);
    const Sxx = new Float64Array(half + 1);
    const Syy = new Float64Array(half + 1);
    const SxyRe = new Float64Array(half + 1);
    const SxyIm = new Float64Array(half + 1);

    for (let start = 0; start + segLen <= x.length; start += half) {
        // 50 % overlap
        const xRe = new Float64Array(segLen),
            xIm = new Float64Array(segLen);
        const yRe = new Float64Array(segLen),
            yIm = new Float64Array(segLen);
        for (let k = 0; k < segLen; k++) {
            xRe[k] = x[start + k] * win[k];
            yRe[k] = y[start + k] * win[k];
        }
        fftInPlace(xRe, xIm);
        fftInPlace(yRe, yIm);
        for (let k = 0; k <= half; k++) {
            Sxx[k] += xRe[k] * xRe[k] + xIm[k] * xIm[k];
            Syy[k] += yRe[k] * yRe[k] + yIm[k] * yIm[k];
            // Sxy = X * conj(Y)
            SxyRe[k] += xRe[k] * yRe[k] + xIm[k] * yIm[k];
            SxyIm[k] += xIm[k] * yRe[k] - xRe[k] * yIm[k];
        }
    }

    const coh = new Float64Array(half + 1);
    for (let k = 0; k <= half; k++) {
        const denom = Sxx[k] * Syy[k];
        if (denom > 0) coh[k] = (SxyRe[k] * SxyRe[k] + SxyIm[k] * SxyIm[k]) / denom;
    }
    return coh;
}

/**
 * Return the expected gain-crossover search window (Hz) for the given prop size.
 */
function _getCrossoverSearchWindow(propInches) {
    const MAP = {
        3: { min: 30, max: 100 },
        4: { min: 25, max: 80 },
        5: { min: 20, max: 70 },
        6: { min: 15, max: 55 },
        7: { min: 10, max: 50 },
        8: { min: 8, max: 40 },
        9: { min: 6, max: 32 },
        10: { min: 5, max: 30 },
    };
    return MAP[propInches] ?? { min: 5, max: 100 };
}

/**
 * Compute stability margins from arrays of frequencies, magnitudes and phases.
 * freqAxis: Hz,  magDB: dB,  phaseDeg: degrees (unwrapped, for Bode plot only).
 * rawPhaseDeg: raw atan2 phase in degrees (-180°, +180°] — used for margin numbers.
 * searchWindow: { min, max } Hz — only search within this band.
 * Returns { phaseMargin, gainMargin, gcFreq, pcFreq } — any may be null.
 */
function _computeStabilityMargins(freqAxis, magDB, phaseDeg, rawPhaseDeg, searchWindow, coherence, axisName) {
    const winMin = searchWindow?.min ?? 5;
    const winMax = searchWindow?.max ?? 100;

    let gcFreq = null,
        phaseMargin = null;
    let pcFreq = null,
        gainMargin = null;
    let gcCoherenceLow = false;
    let phaseMarginInvalid = false;

    // Gain crossover: descending 0 dB crossing within window, searched HIGH→LOW
    // Collect indices inside the window first, then iterate in reverse.
    const winIndices = [];
    for (let i = 0; i < freqAxis.length; i++) {
        if (freqAxis[i] >= winMin && freqAxis[i] <= winMax) winIndices.push(i);
    }
    for (let k = winIndices.length - 1; k >= 1; k--) {
        const i = winIndices[k];
        const j = winIndices[k - 1];
        // Descending crossing: previous bin >= 0, current bin < 0
        if (magDB[j] >= 0 && magDB[i] < 0) {
            const t = magDB[j] / (magDB[j] - magDB[i]);
            gcFreq = freqAxis[j] + t * (freqAxis[i] - freqAxis[j]);
            // Phase margin: use raw atan2 at bin j directly — NOT interpolated.
            // atan2Result = Math.atan2(hIm, hRe) * 180 / Math.PI at the crossover bin.
            const rawJ = rawPhaseDeg[j];
            const atan2Result = rawJ;
            phaseMargin = 180 + atan2Result;

            // Debug logging — verify atan2 value and phase margin formula
            const axis = axisName ?? "unknown";
            console.log(
                `[AeroTune] axis=${axis} atan2=${atan2Result.toFixed(2)}°` +
                    ` phaseMargin=180+(${atan2Result.toFixed(2)})=${phaseMargin.toFixed(2)}°`,
            );

            // Phase margin must be in [0°, 180°] — null (not clamp) if outside
            if (phaseMargin < 0 || phaseMargin > 180) {
                console.warn(
                    `[AeroTune] ${axis}: phase margin ${phaseMargin.toFixed(1)}° outside valid range [0°, 180°] — marking unreliable.`,
                );
                phaseMarginInvalid = true;
                phaseMargin = null;
            }

            // Coherence check at crossover bin
            if (coherence && coherence[j] < 0.5) {
                gcCoherenceLow = true;
            }

            break;
        }
    }

    // Phase crossover: first downward raw -180° crossing within window.
    // Handles both a direct descent through -180° and a wrap-around discontinuity
    // (raw phase jumps from near -180° to near +180° as the unwrapped phase crosses -180°).
    for (let k = 1; k < winIndices.length; k++) {
        const i = winIndices[k];
        const j = winIndices[k - 1];
        // Direct crossing: raw phase descends through -180°
        if (rawPhaseDeg[j] >= -180 && rawPhaseDeg[i] < -180) {
            const t = (rawPhaseDeg[j] + 180) / (rawPhaseDeg[j] - rawPhaseDeg[i]);
            pcFreq = freqAxis[j] + t * (freqAxis[i] - freqAxis[j]);
            gainMargin = -(magDB[j] + t * (magDB[i] - magDB[j]));
            break;
        }
        // Wrap-around crossing: raw phase jumps from near -180° to near +180°.
        // Interpolate using the effective unwrapped value at bin i (rawPhaseDeg[i] - 360).
        if (rawPhaseDeg[j] < -90 && rawPhaseDeg[i] > 90) {
            const t = (-180 - rawPhaseDeg[j]) / (rawPhaseDeg[i] - 360 - rawPhaseDeg[j]);
            pcFreq = freqAxis[j] + t * (freqAxis[i] - freqAxis[j]);
            gainMargin = -(magDB[j] + t * (magDB[i] - magDB[j]));
            break;
        }
    }

    return { phaseMargin, gainMargin, gcFreq, pcFreq, gcCoherenceLow, phaseMarginInvalid };
}

/**
 * Synthesise PID adjustments from stability margins using fixed prop-size baselines.
 * baselineHz: prop-size midpoint of the crossover search window.
 * propInches: prop size in inches — selects the fixed P/D baseline, not current FC PIDs.
 *
 * P: scaleFactor = PM / 45 (clamped 0.7–1.5) applied to fixed prop-size baseP.
 *   Result is independent of current FC PIDs — same flight data always yields same answer.
 *
 * D: fixed prop-size baseD, then:
 *   PM < 40° → +25% (add damping).
 *   PM 40–50° → unchanged.
 *   PM > 50° AND gcFreq > baseline → +15%.
 *
 * Returns { suggestP, suggestD, direction, reason }.
 */
function _synthesizePID(currentP, currentD, margins, baselineHz, propInches) {
    const TARGET_PM = 45;
    const baseline = baselineHz ?? 80;
    const prop = propInches ?? 5;

    // Fixed prop-size baseline tables — independent of current FC PIDs
    const P_BASELINE = { 3: 50, 4: 47, 5: 45, 6: 42, 7: 38, 8: 35, 9: 32, 10: 28 };
    const D_BASELINE = { 3: 28, 4: 25, 5: 23, 6: 21, 7: 18, 8: 16, 9: 14, 10: 12 };
    const propKey = Math.round(prop);
    const baseP = P_BASELINE[propKey] ?? 45;
    const baseD = D_BASELINE[propKey] ?? 23;

    let suggestP = baseP;
    let suggestD = baseD;
    let direction = "=";
    const parts = [];

    if (margins.phaseMargin !== null) {
        const pm = margins.phaseMargin;
        // scaleFactor applied to fixed baseline — not current FC P
        const scaleFactor = clamp(pm / TARGET_PM, 0.7, 1.5);
        suggestP = Math.round(baseP * scaleFactor);
        direction = scaleFactor > 1.01 ? "↑" : scaleFactor < 0.99 ? "↓" : "=";
        parts.push(`baseline ${prop}" P=${baseP}, scale ×${scaleFactor.toFixed(2)} from PM ${pm.toFixed(1)}°`);
    }

    if (margins.phaseMargin !== null) {
        const pm = margins.phaseMargin;
        if (pm < 40) {
            // Under-damped — add damping aggressively
            suggestD = Math.round(baseD * 1.25);
            parts.push(`D +25% (low PM — add damping)`);
        } else if (pm > 50 && margins.gcFreq !== null && margins.gcFreq > baseline) {
            // Stable with high GC frequency — small D increase is safe
            suggestD = Math.round(baseD * 1.15);
            parts.push(`D +15% (GC ${margins.gcFreq.toFixed(0)}Hz > baseline)`);
        }
        // PM 40–50°: D unchanged
    }

    return { suggestP, suggestD, direction, reason: parts.join("; ") || "within target margins" };
}

/**
 * Run the full SysID pipeline on decoded frames for one BBL session.
 * propInches: prop size in inches (from chirpPropInch selector) — used for crossover search window.
 * Returns a sysidResult object or null on fatal error.
 */
function runSysID(frames, config, propInches) {
    const looptime = config.misc?.looptime;
    if (!looptime || looptime <= 0) return null;
    const sampleRate = 1e6 / looptime;

    const AXES = [
        { name: "roll", spKey: "setpoint[0]", gyroKey: "gyroADC[0]", pidKey: "roll" },
        { name: "pitch", spKey: "setpoint[1]", gyroKey: "gyroADC[1]", pidKey: "pitch" },
        { name: "yaw", spKey: "setpoint[2]", gyroKey: "gyroADC[2]", pidKey: "yaw" },
    ];

    const searchWindow = _getCrossoverSearchWindow(propInches);
    const baselineHz = (searchWindow.min + searchWindow.max) / 2;
    const result = { axes: {}, warnings: [], searchWindow, propInches: propInches ?? null };

    for (const ax of AXES) {
        // Extract raw signal arrays
        const sp = new Float64Array(frames.length);
        const gy = new Float64Array(frames.length);
        for (let i = 0; i < frames.length; i++) {
            sp[i] = Number(frames[i][ax.spKey] ?? 0);
            gy[i] = Number(frames[i][ax.gyroKey] ?? 0);
        }

        // Find the longest active chirp segment (rolling 500-sample window, std > 30)
        const { start, end } = _findLongestActiveSegment(sp, 500, 30);
        const N = end - start;

        if (N < 512) {
            result.axes[ax.name] = { error: "Insufficient chirp data on this axis (< 512 samples active)" };
            continue;
        }

        const spSeg = sp.slice(start, end);
        const gySeg = gy.slice(start, end);

        // Pad to next power of 2 and apply Hann window
        const Np = _nextPow2(N);
        const spRe = new Float64Array(Np),
            spIm = new Float64Array(Np);
        const gyRe = new Float64Array(Np),
            gyIm = new Float64Array(Np);
        const win = _hannWindow(N);
        for (let i = 0; i < N; i++) {
            spRe[i] = spSeg[i] * win[i];
            gyRe[i] = gySeg[i] * win[i];
        }

        fftInPlace(spRe, spIm);
        fftInPlace(gyRe, gyIm);

        // Frequency axis: bin k → k * sampleRate / Np Hz
        const nBins = Np / 2 + 1;
        const binHz = sampleRate / Np;

        // H(f) = FFT(gyro) / FFT(setpoint)
        const magDB = new Float64Array(nBins);
        const phaseRad = new Float64Array(nBins);
        for (let k = 0; k < nBins; k++) {
            const spMag2 = spRe[k] * spRe[k] + spIm[k] * spIm[k];
            if (spMag2 < 1e-10) {
                magDB[k] = -60;
                continue;
            }
            const hRe = (gyRe[k] * spRe[k] + gyIm[k] * spIm[k]) / spMag2;
            const hIm = (gyIm[k] * spRe[k] - gyRe[k] * spIm[k]) / spMag2;
            const hMag = Math.sqrt(hRe * hRe + hIm * hIm);
            magDB[k] = hMag > 0 ? 20 * Math.log10(hMag) : -60;
            phaseRad[k] = Math.atan2(hIm, hRe);
        }

        // Snapshot raw atan2 phase BEFORE unwrapping — used for margin numbers
        const phaseRadRaw = phaseRad.slice();

        _unwrapPhase(phaseRad);

        // Coherence via Welch (512-sample segments, 50% overlap)
        const SEG = 512;
        const cohRaw = _welchCoherence(Array.from(spSeg), Array.from(gySeg), SEG);
        const cohBinHz = sampleRate / SEG;

        // Build filtered arrays: 1–500 Hz range only
        const FREQ_MIN = 1,
            FREQ_MAX = 500;
        const freqAxis = [],
            filtMag = [],
            filtPhase = [],
            rawPhaseDeg = [],
            filtCoh = [];
        for (let k = 0; k < nBins; k++) {
            const f = k * binHz;
            if (f < FREQ_MIN || f > FREQ_MAX) continue;
            freqAxis.push(f);
            filtMag.push(magDB[k]);
            filtPhase.push((phaseRad[k] * 180) / Math.PI); // unwrapped — for Bode plot drawing
            rawPhaseDeg.push((phaseRadRaw[k] * 180) / Math.PI); // raw atan2 — for margin numbers
            // Map coherence bin: interpolate from Welch resolution
            const ck = f / cohBinHz;
            const ci = Math.floor(ck);
            const cf = ck - ci;
            const c =
                ci + 1 < cohRaw.length
                    ? cohRaw[ci] * (1 - cf) + cohRaw[ci + 1] * cf
                    : ci < cohRaw.length
                      ? cohRaw[ci]
                      : 0;
            filtCoh.push(Math.min(1, Math.max(0, c)));
        }

        // Stability margins — only search within the prop-size-specific window
        const { phaseMargin, gainMargin, gcFreq, pcFreq, gcCoherenceLow, phaseMarginInvalid } =
            _computeStabilityMargins(
                freqAxis,
                filtMag,
                filtPhase,
                rawPhaseDeg,
                searchWindow,
                filtCoh,
                ax.name.toUpperCase(),
            );

        // Current PID values from BBL header
        const currentP = config.pids?.[ax.pidKey]?.[0] ?? null;
        const currentD = config.pids?.[ax.pidKey]?.[2] ?? null;

        const pidSuggest = !phaseMarginInvalid
            ? _synthesizePID(currentP, currentD, { phaseMargin, gainMargin, gcFreq, pcFreq }, baselineHz, propInches)
            : null;

        const currentI = config.pids?.[ax.pidKey]?.[1] ?? null;

        result.axes[ax.name] = {
            freqAxis,
            magDB: filtMag,
            phaseDeg: filtPhase,
            coherence: filtCoh,
            phaseMargin,
            phaseMarginInvalid,
            gainMargin,
            gcFreq,
            pcFreq,
            currentP,
            currentI,
            currentD,
            pidSuggest,
            spSeg,
            gySeg,
            sampleRate,
        };

        // Safety warnings
        if (gcCoherenceLow) {
            result.warnings.push(`${ax.name.toUpperCase()}: Low coherence at crossover — margin may be unreliable.`);
        }
        if (phaseMargin !== null) {
            if (phaseMargin < 30) {
                result.warnings.push(
                    `${ax.name.toUpperCase()}: Phase margin ${phaseMargin.toFixed(1)}° is dangerously close to instability (<30°) — reduce P gain immediately.`,
                );
            } else if (phaseMargin > 70) {
                result.warnings.push(
                    `${ax.name.toUpperCase()}: Phase margin ${phaseMargin.toFixed(1)}° is high (>70°) — good stability margin at hover — safe to increase P slightly and re-test with aggressive flight data.`,
                );
            }
        }
    }

    return result;
}

// ─────────────────────────────────────────────────────────────────────────────
// BBL helpers
// ─────────────────────────────────────────────────────────────────────────────

const BBL_MARKER_BYTES = Array.from("H Product:Blackbox flight data recorder by Nicholas Sherlock").map((c) =>
    c.codePointAt(0),
);

// Returns the byte offset of the Nth session's product-marker line (-1 if not found).
function findSessionMarkerPos(buf, sessionIndex) {
    const len = buf.length;
    let pos = 0;
    let sessionsFound = -1;

    while (pos < len) {
        if (buf[pos] === BBL_MARKER_BYTES[0]) {
            let isMarker = BBL_MARKER_BYTES.length + pos <= len;
            for (let j = 1; isMarker && j < BBL_MARKER_BYTES.length; j++) {
                if (buf[pos + j] !== BBL_MARKER_BYTES[j]) {
                    isMarker = false;
                }
            }
            if (isMarker) {
                sessionsFound++;
                if (sessionsFound === sessionIndex) {
                    return pos;
                }
            }
        }
        pos++;
    }
    return -1;
}

// Returns the byte offset where binary frame data begins for a given session,
// i.e. the position immediately after the last 'H ...' header line of that session.
function findBBLBinaryStart(buf, sessionIndex = 0) {
    const sessionHeaderStart = findSessionMarkerPos(buf, sessionIndex);
    if (sessionHeaderStart === -1) {
        return 0;
    } // session not found

    // Phase 2: scan forward from the session marker, collecting 'H ' lines.
    // The first non-'H ' line marks the start of binary data.
    const len = buf.length;
    let pos = sessionHeaderStart;
    let lastHeaderEnd = 0;

    while (pos < len) {
        const lineStart = pos;
        while (pos < len && buf[pos] !== 0x0a) {
            pos++;
        } // find \n
        if (pos < len) {
            pos++;
        } // skip \n

        if (pos - lineStart < 2) {
            continue;
        }

        if (buf[lineStart] === 0x48 && buf[lineStart + 1] === 0x20) {
            // 'H ' line — still in header
            lastHeaderEnd = pos;
        } else {
            break; // binary data starts here
        }
    }

    return lastHeaderEnd;
}

// Returns the byte offset of the Nth session's product-marker line.
// Used to compute where session N's binary data must end.
function findBBLSessionHeaderStart(buf, sessionIndex) {
    return findSessionMarkerPos(buf, sessionIndex);
}

// ─────────────────────────────────────────────────────────────────────────────
// Vue Component
// ─────────────────────────────────────────────────────────────────────────────
const STORAGE_KEY = "aerotune_inputs";

function loadStoredInputs() {
    try {
        const raw = localStorage.getItem(STORAGE_KEY);
        return raw ? JSON.parse(raw) : null;
    } catch {
        return null;
    }
}

export default {
    name: "AeroTuneTab",
    components: { BaseTab },

    setup() {
        return { pidTuningStore: usePidTuningStore() };
    },

    data() {
        const stored = loadStoredInputs();
        return {
            activeView: "calculator",
            // Calculator inputs — restored from localStorage if available
            kv: stored?.kv ?? 2400,
            voltage: stored?.voltage ?? 22.2,
            prop: stored?.prop ?? 5,
            weight: stored?.weight ?? 500,
            style: stored?.style ?? "Bando",
            voltagePresets: [
                { label: "1S", v: 3.7 },
                { label: "2S", v: 7.4 },
                { label: "3S", v: 11.1 },
                { label: "4S", v: 14.8 },
                { label: "5S", v: 18.5 },
                { label: "6S", v: 22.2 },
                { label: "8S", v: 29.6 },
            ],
            // Calculator outputs
            showResults: false,
            pids: {},
            filterRec: { hz: "--", low: "--", high: "--", note: "" },
            copyBtnText: "📋 COPY ALL VALUES",
            sysidCopyBtnText: "📋 COPY ALL VALUES",
            // Analyzer
            motorTemp: "WARM",
            csvFile: null,
            fileName: "No file selected",
            analysisResult: "Select a Betaflight blackbox file (.bfl, .bbl, or .csv) and click ANALYZE.",
            // Multi-session BBL support
            bblSessions: [],
            bblSelectedSession: 0,
            bblBuffer: null,
            spectrogramVisible: false,
            graphsVisible: false,
            graphAxes: [
                { name: "roll", label: "Roll", color: "#e74c3c" },
                { name: "pitch", label: "Pitch", color: "#3498db" },
                { name: "yaw", label: "Yaw", color: "#2ecc71" },
            ],
            graphToggles: {
                gyro: { roll: true, pitch: true, yaw: false },
                setpoint: { roll: true, pitch: true, yaw: false },
                pidError: { roll: true, pitch: true, yaw: false },
            },
            graphZoomLevels: { gyro: 1, setpoint: 1, pidError: 1, spectrogram: 1 },
            graphPanOffsets: { gyro: 0, setpoint: 0, pidError: 0, spectrogram: 0 },
            spectrogramGain: 100,
            _graphFrames: null,
            _graphConfig: null,
            extendedAnalysis: null,
            logPidOutput: null,
            logPidCopyBtnText: "📋 COPY NEW PIDs",
            sysidResult: null,
            sysidActiveAxis: "roll",
            sysidZoom: "full",
            tooltip: { visible: false, text: "", x: 0, y: 0 },
            // Auto Tune (chirp sweep)
            chirpPropInch: 5,
            chirpPitch: 230,
            chirpRoll: 230,
            chirpYaw: 230,
            chirpPitchLevel: "MEDIUM",
            chirpRollLevel: "MEDIUM",
            chirpYawLevel: "MEDIUM",
            chirpStartHz: 80,
            chirpEndHz: 600,
            chirpDuration: 10,
            chirpConfigured: false,
            chirpConfirmText: "",
            advancedOpen: false,
        };
    },

    computed: {
        canApply() {
            return this.showResults && CONFIGURATOR.connectionValid;
        },
        workflowInstructions() {
            return "1. Calculate baseline PIDs · 2. Configure Blackbox · 3. Fly the test pattern · 4. Load your .bfl file and Analyze";
        },
        sysidPids() {
            if (!this.sysidResult) return null;
            const axIdxMap = { roll: 0, pitch: 1, yaw: 2 };
            const ffKeys = { roll: "feedforwardRoll", pitch: "feedforwardPitch", yaw: "feedforwardYaw" };
            const dMaxKeys = { roll: "dMaxRoll", pitch: "dMaxPitch", yaw: null };
            const result = {};
            for (const ax of ["roll", "pitch", "yaw"]) {
                const axData = this.sysidResult.axes[ax];
                const axIdx = axIdxMap[ax];
                const hasSuggest = axData && !axData.error && !axData.phaseMarginInvalid && axData.pidSuggest;
                const suggestP = hasSuggest ? axData.pidSuggest.suggestP : null;
                const suggestD = hasSuggest ? axData.pidSuggest.suggestD : null;
                // I: from BBL header if available, else from live FC state
                const currentI = axData?.currentI ?? FC.PIDS?.[axIdx]?.[1] ?? "?";
                // FF: from live FC advanced tuning if connected
                const ffKey = ffKeys[ax];
                const currentFF = FC.ADVANCED_TUNING?.[ffKey] ?? "?";
                const dMaxKey = dMaxKeys[ax];
                const existingDMax =
                    dMaxKey && FC.ADVANCED_TUNING?.[dMaxKey] != null ? FC.ADVANCED_TUNING[dMaxKey] : null;
                result[ax] = {
                    P: suggestP ?? axData?.currentP ?? "—",
                    I: currentI,
                    D: suggestD ?? (ax !== "yaw" ? (axData?.currentD ?? "—") : "—"),
                    DMax: suggestD != null ? suggestD + 3 : (existingDMax ?? "—"),
                    FF: currentFF,
                    reason: hasSuggest ? axData.pidSuggest.reason : null,
                };
            }
            return result;
        },
        canApplySysID() {
            return !!this.sysidResult && CONFIGURATOR.connectionValid;
        },
        canApplyLogPids() {
            return !!this.logPidOutput && CONFIGURATOR.connectionValid;
        },
        sysidAvailableAxes() {
            if (!this.sysidResult) return [];
            const defs = [
                { name: "roll", label: "Roll", color: "#ff4444" },
                { name: "pitch", label: "Pitch", color: "#4488ff" },
                { name: "yaw", label: "Yaw", color: "#44cc66" },
            ];
            return defs.filter((d) => this.sysidResult.axes[d.name] && !this.sysidResult.axes[d.name].error);
        },
    },

    watch: {
        kv(v) {
            this._persistInputs();
            this.showResults = false;
            this.pids = {};
        },
        voltage(v) {
            this._persistInputs();
            this.showResults = false;
            this.pids = {};
        },
        prop(v) {
            this._persistInputs();
            this.showResults = false;
            this.pids = {};
        },
        weight(v) {
            this._persistInputs();
            this.showResults = false;
            this.pids = {};
        },
        style(v) {
            this._persistInputs();
            this.showResults = false;
            this.pids = {};
        },
        chirpPropInch(v) {
            this.applyChirpPropDefaults(v);
        },
        sysidResult(val) {
            if (val) {
                // Auto-select the first axis that has valid data
                const axes = ["roll", "pitch", "yaw"];
                this.sysidActiveAxis = axes.find((ax) => val.axes[ax] && !val.axes[ax].error) || "roll";
                this.sysidZoom = "full";
                this.$nextTick(() => this.renderChirpOverlay());
            }
        },
    },

    methods: {
        showTip(event, text) {
            const rect = event.currentTarget.getBoundingClientRect();
            this.tooltip.text = text;
            this.tooltip.x = rect.left + rect.width / 2;
            this.tooltip.y = rect.top - 8;
            this.tooltip.visible = true;
        },
        hideTip() {
            this.tooltip.visible = false;
        },
        _persistInputs() {
            try {
                localStorage.setItem(
                    STORAGE_KEY,
                    JSON.stringify({
                        kv: this.kv,
                        voltage: this.voltage,
                        prop: this.prop,
                        weight: this.weight,
                        style: this.style,
                    }),
                );
            } catch {
                /* storage unavailable — silently ignore */
            }
        },
        openInstructionsPopup() {
            // Build the HTML as a Blob and open via object URL to avoid
            // document.write() (flagged as a security hotspot by static analysis).
            const html = `<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>AeroTune – Instructions</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    background: #2e2e2e;
    color: #cccccc;
    font-family: 'Open Sans', Arial, sans-serif;
    font-size: 13px;
    line-height: 1.7;
    padding: 24px 28px 40px;
  }
  h1 { font-size: 16px; color: #ffffff; margin-bottom: 4px; }
  .subtitle { font-size: 11px; color: #888888; margin-bottom: 20px; }
  .subtitle a { color: #ffbb00; }
  h3 {
    color: #ffbb00;
    font-size: 13px;
    margin: 20px 0 6px;
    border-bottom: 1px solid #444444;
    padding-bottom: 4px;
  }
  p, li { font-size: 12px; color: #aaaaaa; line-height: 1.7; margin: 4px 0; }
  ul { padding-left: 20px; }
  code {
    background: #1a1a1a;
    padding: 1px 5px;
    border-radius: 2px;
    font-size: 11px;
    color: #cccccc;
  }
  a { color: #ffbb00; }
  strong { color: #dddddd; }
  .ok   { color: #00d966; }
  .warn { color: #ffe66d; }
  .bad  { color: #ff6b6b; }
</style>
</head>
<body>
<h1>AeroTune™ Instructions</h1>
<h3>STEP 1: CALCULATE BASELINE PIDs</h3>
<ul>
  <li>Enter motor KV, battery voltage, prop size, weight and flying style.</li>
  <li>Click <strong>CALCULATE PIDs</strong> to get conservative baseline values.</li>
  <li>Click <strong>APPLY PIDs TO FC</strong> to write them directly to the PID Tuning tab, or use <strong>COPY ALL VALUES</strong> to copy them to the clipboard.</li>
</ul>

<h3>STEP 2: CONFIGURE BLACKBOX</h3>
<ul>
  <li>In Betaflight, go to Configuration → Blackbox → Enable, Device = SD Card.</li>
  <li>Set Blackbox logging rate to 1/2 or better — higher rates give better frequency resolution for the analyzer.</li>
  <li>Enable: Gyro, Gyro (Unfiltered), Motor, PID, RC Commands, RPM, Setpoint, Accelerometer.</li>
  <li><strong>Betaflight 4.5+:</strong> raw gyro is always logged automatically.</li>
  <li><strong>Betaflight 4.3/4.4:</strong> set Debug Mode to <code>GYRO_SCALED</code> to capture unfiltered gyro data.</li>
  <li>Use fresh propellers — damaged props introduce false noise and will give inaccurate results.</li>
</ul>

<h3>STEP 3: FLY THE TEST PATTERN</h3>
<ul>
  <li>Level mode or Acro mode both work — LOS or FPV.</li>
  <li>Level mode: full left stick hold 1–1.5 seconds, pause, full right, pause, full forward, pause, full back.</li>
  <li>Acro mode: sharp direct inputs at 20° and 45°, with brief pauses between each.</li>
  <li>Aim for a 2 minute flight. Fly through the full throttle range — the Analyzer needs data across all throttle levels to give an accurate result.</li>
</ul>

<h3>STEP 4: ANALYZE THE LOG</h3>
<ul>
  <li>In Betaflight, go to the Blackbox tab and click USB Storage Mode.</li>
  <li>Drag your .bfl file from the FC storage to your desktop.</li>
  <li>Unplug the FC, then plug it back in and open Betaflight.</li>
  <li>In the AeroTune tab, click Select BBL / BFL, choose your .bfl file and click ANALYZE.</li>
</ul>

<h3>INTERPRETING RESULTS</h3>
<ul>
  <li><span class="ok">EXCELLENT / CLEAN</span> – filters are well-tuned, no changes needed</li>
  <li><span class="ok">GOOD</span> – minor adjustments may help</li>
  <li><span class="warn">FAIR</span> – lower Gyro Lowpass 2 by ~30 Hz, re-test</li>
  <li><span class="warn">WEAK</span> – lower by ~50 Hz, consider adding a Notch filter</li>
  <li><span class="bad">VERY WEAK</span> – aggressive filter reduction needed; check for mechanical vibration</li>
</ul>

<h3>FILTER RECOMMENDATION NOTE</h3>
<ul>
  <li><strong>Gyro Lowpass 2:</strong> The value shown in the Calculator is a <em>starting point</em> based on prop size. Use the Analyzer results to fine-tune after flying.</li>
  <li><strong>Dynamic Notch Filter:</strong> Enable and adjust count based on analyzer results to target resonant frequencies.</li>
  <li><strong>Gyro RPM Filter:</strong> Enable if using bidirectional DSHOT — the most effective filter available for eliminating motor noise harmonics.</li>
</ul>
</body>
</html>`;
            const blob = new Blob([html], { type: "text/html" });
            const url = URL.createObjectURL(blob);
            globalThis.open(url, "aerotune_instructions", "width=620,height=800,resizable=yes,scrollbars=yes");
        },

        selectVoltage(v) {
            this.voltage = v;
        },

        calculate() {
            const pids = calculatePIDs(this.kv, this.voltage, this.prop, this.weight, this.style);
            if (!pids) {
                alert("Invalid input values. Please check all fields.");
                return;
            }
            this.pids = pids;
            this.filterRec = filterRecommendation(this.prop);
            this.showResults = true;
        },

        async applyToFC() {
            if (!this.showResults || !CONFIGURATOR.connectionValid) {
                alert("No flight controller connected. Connect to FC before applying PIDs.");
                return;
            }

            const p = this.pids;

            // Read current PID values from FC before patching so we write
            // against the live FC state, not stale defaults.
            try {
                await MSP.promise(MSPCodes.MSP_PID);
            } catch (e) {
                console.error("[AeroTune] Failed to read MSP_PID before applying:", e);
                alert("Failed to read PID values from FC. Check connection and try again.");
                return;
            }

            // Write into FC reactive state
            if (FC.PIDS && FC.PIDS.length >= 3) {
                FC.PIDS[0][0] = p.roll_p;
                FC.PIDS[0][1] = p.roll_i;
                FC.PIDS[0][2] = p.roll_d;
                FC.PIDS[1][0] = p.pitch_p;
                FC.PIDS[1][1] = p.pitch_i;
                FC.PIDS[1][2] = p.pitch_d;
                FC.PIDS[2][0] = p.yaw_p;
                FC.PIDS[2][1] = p.yaw_i;
                FC.PIDS[2][2] = p.yaw_d;
            }

            // Read current advanced tuning values from FC before modifying so
            // we preserve ALL fields (idleMinRpm, TPA, iterm relax, anti-gravity,
            // etc.) that AeroTune doesn't touch.  Without this read, FC.ADVANCED_TUNING
            // may still be at its all-zero defaults (if the user hasn't visited
            // the PID tab yet), which would zero out those settings on the FC.
            try {
                await MSP.promise(MSPCodes.MSP_PID_ADVANCED);
            } catch (e) {
                console.error("[AeroTune] Failed to read MSP_PID_ADVANCED before applying:", e);
                alert("Failed to read advanced tuning from FC. Check connection and try again.");
                return;
            }

            // Now patch only the feedforward and D Max fields.
            if (FC.ADVANCED_TUNING) {
                FC.ADVANCED_TUNING.feedforwardRoll = p.roll_f;
                FC.ADVANCED_TUNING.feedforwardPitch = p.pitch_f;
                FC.ADVANCED_TUNING.feedforwardYaw = p.yaw_f;
                FC.ADVANCED_TUNING.dMaxRoll = p.dMax_roll;
                FC.ADVANCED_TUNING.dMaxPitch = p.dMax_pitch;
            }

            // Push to FC hardware RAM so PID tab reads back the new values on mount
            try {
                await MSP.promise(MSPCodes.MSP_SET_PID, mspHelper.crunch(MSPCodes.MSP_SET_PID));
                await MSP.promise(MSPCodes.MSP_SET_PID_ADVANCED, mspHelper.crunch(MSPCodes.MSP_SET_PID_ADVANCED));
            } catch (e) {
                console.error("[AeroTune] Failed to send PID values to FC:", e);
                alert("Failed to send PID values to FC. Check connection and try again.");
                return;
            }

            // Tell the PID Tuning store that values were written externally so
            // the Save button is enabled when the tab loads the data.
            this.pidTuningStore.markExternalChange();

            // Navigate to PID tuning tab
            const pidTabLink = document.querySelector("li.tab_pid_tuning a");
            if (pidTabLink) {
                pidTabLink.click();
            } else {
                alert("PID Tuning tab not available. Make sure a flight controller is connected.");
            }
        },

        copyValues() {
            if (!this.showResults) {
                return;
            }
            const p = this.pids,
                fr = this.filterRec;
            const text = [
                `# AeroTune V5.6 PID Values`,
                `Roll   P=${p.roll_p}  I=${p.roll_i}  D_Max=${p.dMax_roll}  F=${p.roll_f}  D_min=${p.d_min_roll}`,
                `Pitch  P=${p.pitch_p}  I=${p.pitch_i}  D_Max=${p.dMax_pitch}  F=${p.pitch_f}  D_min=${p.d_min_pitch}`,
                `Yaw    P=${p.yaw_p}  I=${p.yaw_i}  D=${p.yaw_d}  F=${p.yaw_f}`,
                `Gyro Lowpass 2 recommendation: ${fr.hz} Hz (${fr.low}–${fr.high} Hz) – ${fr.note}`,
            ].join("\n");
            navigator.clipboard
                .writeText(text)
                .then(() => {
                    this.copyBtnText = "✔ Copied!";
                    setTimeout(() => {
                        this.copyBtnText = "📋 COPY ALL VALUES";
                    }, 2000);
                })
                .catch((err) => {
                    console.error("[AeroTune] Failed to copy to clipboard:", err);
                });
        },

        /** Decode and analyze a specific BBL session from the already-loaded buffer. */
        _decodeBBLSession(sessionIdx, buffer, sessions) {
            const config = sessions[sessionIdx];
            const headerEnd = findBBLBinaryStart(buffer, sessionIdx);
            if (headerEnd === 0) {
                this.analysisResult = "ERROR: Could not locate frame data in blackbox file.";
                return;
            }

            // Bound the decode to this session's byte range so multi-session
            // logs don't bleed into the next session's header bytes.
            const nextHeaderStart =
                sessionIdx + 1 < sessions.length ? findBBLSessionHeaderStart(buffer, sessionIdx + 1) : -1;
            const sessionEnd = nextHeaderStart >= 0 ? nextHeaderStart : buffer.length;

            const decoder = new FrameDecoder(config);
            const { frames } = decoder.decodeFrames(buffer, headerEnd, 0, sessionEnd);
            if (!frames || frames.length === 0) {
                this.analysisResult =
                    "ERROR: No frames decoded from blackbox file. The file may be corrupt or use an unsupported format.";
                return;
            }

            // Detect chirp / SysID log — if found, run frequency response analysis
            // and skip the normal filter effectiveness scoring.
            this.sysidResult = null;
            if (detectChirp(frames, config)) {
                const prefix = sessions.length > 1 ? `Session ${sessionIdx + 1} — ` : "";
                this.analysisResult = `${prefix}CHIRP / SYSID log detected — see frequency response analysis below.`;
                try {
                    this.sysidResult = runSysID(frames, config, this.chirpPropInch);
                } catch (sysidErr) {
                    this.analysisResult += `\nSysID analysis error: ${sysidErr.message}`;
                }
                return;
            }

            const prefix = sessions.length > 1 ? `Session ${sessionIdx + 1}: ` : "";
            const result = analyzeLog(frames, this.motorTemp, config);
            this.analysisResult = prefix + formatAnalysisResult(result);
            this._processAnalysisGraphs(frames, config, result);
        },

        /** Called by the session dropdown — re-analyzes the selected session. */
        runBBLSession(sessionIdx) {
            if (!this.bblBuffer || !this.bblSessions.length) {
                return;
            }
            try {
                this._decodeBBLSession(sessionIdx, this.bblBuffer, this.bblSessions);
            } catch (err) {
                this.analysisResult = `ERROR: Failed to decode session ${sessionIdx + 1}: ${err.message}`;
            }
        },

        onFileChange(e) {
            const file = e.target.files[0];
            if (!file) {
                return;
            }
            this.bblBuffer = null;
            this.bblSessions = [];
            this.bblSelectedSession = 0;
            this.sysidResult = null;
            this.csvFile = file;
            this.fileName = file.name;
        },

        async analyzeFile() {
            if (!this.csvFile) {
                return;
            }
            this.analysisResult = "Parsing file…";
            const motorTemp = this.motorTemp;
            const file = this.csvFile;
            const ext = file.name.split(".").pop().toLowerCase();

            try {
                if (ext === "bfl" || ext === "bbl") {
                    const arrayBuf = await file.arrayBuffer();
                    const buffer = new Uint8Array(arrayBuf);

                    // Parse ASCII header section — may contain multiple sessions
                    const headerParser = new BBLHeaderParser();
                    const sessions = headerParser.parseFile(buffer);
                    if (!sessions || sessions.length === 0) {
                        this.analysisResult =
                            "ERROR: Could not parse blackbox header. Make sure this is a valid Betaflight blackbox file.";
                        return;
                    }

                    // Store for re-use when the user switches sessions
                    this.bblBuffer = buffer;
                    this.bblSessions = sessions;
                    this.bblSelectedSession = 0;

                    if (sessions.length > 1) {
                        this.analysisResult = `Found ${sessions.length} flight sessions. Showing Session 1 — use the dropdown above to select another.`;
                    }

                    this._decodeBBLSession(0, buffer, sessions);
                } else {
                    // CSV pipeline
                    const text = await file.text();
                    const rows = parseBlackboxCSV(text);
                    if (!rows) {
                        this.analysisResult =
                            "ERROR: Could not find a valid Betaflight blackbox header.\nMake sure you exported a CSV from Blackbox Explorer (not the raw .BFL/.BBL file).";
                        return;
                    }
                    const csvResult = analyzeLog(rows, motorTemp);
                    this.analysisResult = formatAnalysisResult(csvResult);
                    this._processAnalysisGraphs(rows, null, csvResult);
                }
            } catch (err) {
                this.analysisResult = `ERROR: Failed to read file: ${err.message}`;
            }
        },

        _processAnalysisGraphs(frames, config, result) {
            this._graphFrames = frames;
            this._graphConfig = config;
            this.graphsVisible = frames && frames.length > 0;

            // Extended analysis
            const sampleRate = 1e6 / (config?.misc?.looptime ?? 312);
            const motorPoles = config?.motor?.poles ?? 14;
            const stepResponseText = _analyzeStepResponse(frames, sampleRate);

            // Motor heatmap data (merged motor spread + throttle bands)
            const motorHeatmap = _buildMotorHeatmap(frames, motorPoles);

            // I-term bias — compact summary
            const itermBiasText = _analyzeItermBias(frames);
            let itermBiasShort = null;
            if (itermBiasText) {
                const lines = itermBiasText.split("\n").filter((l) => !l.startsWith("  "));
                itermBiasShort = lines.join(" | ");
            }

            this.extendedAnalysis = { stepResponseText, motorHeatmap, itermBiasShort };

            // PID recommendations
            this.logPidOutput = _computeLogPidRecommendations(
                config,
                stepResponseText,
                result.dTermNoise,
                this.motorTemp,
            );

            // Build freq-vs-throttle spectrogram data
            this._freqVsThrottleData = frames.length >= 64 ? _buildFreqVsThrottleData(frames, sampleRate) : null;

            this.$nextTick(() => {
                this.renderGraphs();
            });
        },

        renderGraphs() {
            if (!this._graphFrames || this._graphFrames.length === 0) return;
            const frames = this._graphFrames;
            const config = this._graphConfig;

            // Graph 1: Unfiltered Gyros
            this._renderTimeSeries(this.$refs.graphGyro, frames, {
                fields: [
                    {
                        key: "gyroUnfilt[0]",
                        fallback: "gyroADC[0]",
                        color: "#e74c3c",
                        name: "roll",
                        visible: this.graphToggles.gyro.roll,
                    },
                    {
                        key: "gyroUnfilt[1]",
                        fallback: "gyroADC[1]",
                        color: "#3498db",
                        name: "pitch",
                        visible: this.graphToggles.gyro.pitch,
                    },
                    {
                        key: "gyroUnfilt[2]",
                        fallback: "gyroADC[2]",
                        color: "#2ecc71",
                        name: "yaw",
                        visible: this.graphToggles.gyro.yaw,
                    },
                ],
                zoom: this.graphZoomLevels.gyro,
                pan: this.graphPanOffsets.gyro,
                label: "deg/s",
            });

            // Graph 2: Setpoint (solid) + Gyro (semi-transparent overlay)
            this._renderTimeSeries(this.$refs.graphSetpoint, frames, {
                fields: [
                    { key: "setpoint[0]", color: "#e74c3c", name: "roll-sp", visible: this.graphToggles.setpoint.roll },
                    {
                        key: "gyroADC[0]",
                        color: "#e74c3c",
                        name: "roll-gyro",
                        visible: this.graphToggles.setpoint.roll,
                        alpha: 0.4,
                    },
                    {
                        key: "setpoint[1]",
                        color: "#3498db",
                        name: "pitch-sp",
                        visible: this.graphToggles.setpoint.pitch,
                    },
                    {
                        key: "gyroADC[1]",
                        color: "#3498db",
                        name: "pitch-gyro",
                        visible: this.graphToggles.setpoint.pitch,
                        alpha: 0.4,
                    },
                    { key: "setpoint[2]", color: "#2ecc71", name: "yaw-sp", visible: this.graphToggles.setpoint.yaw },
                    {
                        key: "gyroADC[2]",
                        color: "#2ecc71",
                        name: "yaw-gyro",
                        visible: this.graphToggles.setpoint.yaw,
                        alpha: 0.4,
                    },
                ],
                zoom: this.graphZoomLevels.setpoint,
                pan: this.graphPanOffsets.setpoint,
                label: "deg/s",
            });

            // Graph 3: PID Error (gyroADC - setpoint)
            this._renderTimeSeries(this.$refs.graphPidError, frames, {
                fields: [
                    {
                        key: "gyroADC[0]",
                        subtract: "setpoint[0]",
                        color: "#e74c3c",
                        name: "roll",
                        visible: this.graphToggles.pidError.roll,
                    },
                    {
                        key: "gyroADC[1]",
                        subtract: "setpoint[1]",
                        color: "#3498db",
                        name: "pitch",
                        visible: this.graphToggles.pidError.pitch,
                    },
                    {
                        key: "gyroADC[2]",
                        subtract: "setpoint[2]",
                        color: "#2ecc71",
                        name: "yaw",
                        visible: this.graphToggles.pidError.yaw,
                    },
                ],
                zoom: this.graphZoomLevels.pidError,
                pan: this.graphPanOffsets.pidError,
                label: "error",
            });

            // Graph 4: Freq vs Throttle Spectrogram
            this._renderFreqVsThrottle(this.$refs.graphSpectrogram, config);

            // Legend + Motor heatmap
            this._renderSpectrogramLegend();
            this._renderMotorHeatmap();
        },

        _renderTimeSeries(canvas, frames, opts) {
            if (!canvas) return;
            const ctx = canvas.getContext("2d");
            const W = canvas.width;
            const H = canvas.height;
            const PAD_L = 48,
                PAD_R = 8,
                PAD_T = 4,
                PAD_B = 18;
            const plotW = W - PAD_L - PAD_R;
            const plotH = H - PAD_T - PAD_B;

            // Clear — BF configurator dark surface
            ctx.fillStyle = "hsl(0,0%,8%)";
            ctx.fillRect(0, 0, W, H);

            const zoom = opts.zoom || 1;
            const totalFrames = frames.length;
            const visibleFrames = Math.max(100, Math.floor(totalFrames / zoom));
            const panOffset = Math.min(Math.max(0, opts.pan || 0), Math.max(0, totalFrames - visibleFrames));
            const startFrame = panOffset;
            const endFrame = Math.min(startFrame + visibleFrames, totalFrames);

            // Collect data and find Y range
            let yMin = Infinity,
                yMax = -Infinity;
            const traces = [];
            for (const f of opts.fields) {
                if (!f.visible) continue;
                const vals = [];
                for (let i = startFrame; i < endFrame; i++) {
                    let v = Number(frames[i]?.[f.key] ?? frames[i]?.[f.fallback] ?? 0);
                    if (f.subtract) v -= Number(frames[i]?.[f.subtract] ?? 0);
                    vals.push(v);
                    if (v < yMin) yMin = v;
                    if (v > yMax) yMax = v;
                }
                traces.push({ vals, color: f.color, name: f.name, alpha: f.alpha });
            }

            if (traces.length === 0 || yMin === Infinity) return;

            // Symmetrical Y range
            const yAbs = Math.max(Math.abs(yMin), Math.abs(yMax), 10);
            yMin = -yAbs;
            yMax = yAbs;
            const yRange = yMax - yMin;

            // Grid
            ctx.strokeStyle = "rgba(255,255,255,0.08)";
            ctx.lineWidth = 1;
            ctx.setLineDash([2, 6]);
            // Zero line
            const zeroY = PAD_T + plotH * (yMax / yRange);
            ctx.beginPath();
            ctx.moveTo(PAD_L, zeroY);
            ctx.lineTo(W - PAD_R, zeroY);
            ctx.stroke();
            // Horizontal grid
            const gridSteps = 4;
            for (let g = 1; g <= gridSteps; g++) {
                const frac = g / gridSteps;
                ctx.beginPath();
                ctx.moveTo(PAD_L, PAD_T + plotH * frac);
                ctx.lineTo(W - PAD_R, PAD_T + plotH * frac);
                ctx.stroke();
            }
            ctx.setLineDash([]);

            // Y-axis labels
            ctx.font = "10px monospace";
            ctx.fillStyle = "#666";
            ctx.textBaseline = "middle";
            ctx.textAlign = "right";
            ctx.fillText(`${Math.round(yMax)}`, PAD_L - 4, PAD_T + 6);
            ctx.fillText("0", PAD_L - 4, zeroY);
            ctx.fillText(`${Math.round(yMin)}`, PAD_L - 4, PAD_T + plotH - 6);

            // Draw traces
            const xStep = plotW / (endFrame - startFrame - 1 || 1);
            for (const trace of traces) {
                ctx.strokeStyle = trace.color;
                ctx.lineWidth = trace.alpha ? 1.0 : 1.2;
                ctx.globalAlpha = trace.alpha ?? 0.85;
                ctx.beginPath();
                for (let i = 0; i < trace.vals.length; i++) {
                    const x = PAD_L + i * xStep;
                    const y = PAD_T + plotH * ((yMax - trace.vals[i]) / yRange);
                    if (i === 0) ctx.moveTo(x, y);
                    else ctx.lineTo(x, y);
                }
                ctx.stroke();
                ctx.globalAlpha = 1;
            }

            // Border
            ctx.strokeStyle = "#333";
            ctx.lineWidth = 1;
            ctx.strokeRect(PAD_L, PAD_T, plotW, plotH);
        },

        _renderFreqVsThrottle(canvas, config) {
            if (!canvas || !this._freqVsThrottleData) return;
            const ctx = canvas.getContext("2d");
            const W = canvas.width;
            const H = canvas.height;
            const PAD_L = 48,
                PAD_R = 8,
                PAD_T = 4,
                PAD_B = 22;
            const plotW = W - PAD_L - PAD_R;
            const plotH = H - PAD_T - PAD_B;

            const { matrix, maxBin, maxFreqHz } = this._freqVsThrottleData;

            // Clear — dark background
            ctx.fillStyle = "hsl(0,0%,4%)";
            ctx.fillRect(0, 0, W, H);

            // Find global max for normalisation
            const gainFactor = (this.spectrogramGain || 100) / 100;
            let globalMax = 0;
            for (let t = 0; t < 100; t++) {
                for (let k = 0; k < maxBin; k++) {
                    if (matrix[t][k] > globalMax) globalMax = matrix[t][k];
                }
            }
            if (globalMax === 0) globalMax = 1;

            // Draw heatmap: X=frequency, Y=throttle%
            const img = ctx.createImageData(plotW, plotH);
            for (let px = 0; px < plotW; px++) {
                const freqBin = Math.min(Math.floor((px / plotW) * maxBin), maxBin - 1);
                for (let py = 0; py < plotH; py++) {
                    // py=0 is top=100% throttle, py=plotH-1 is bottom=0% throttle
                    const thrBin = Math.min(99, Math.floor((1 - py / plotH) * 100));
                    const val = Math.min(1, (matrix[thrBin][freqBin] / globalMax) * gainFactor);
                    const idx = (py * plotW + px) * 4;
                    // Hot colormap: black → dark red → red → orange → yellow → white
                    const v4 = val * 4;
                    // Black → dark red → red → orange → yellow → white
                    let r, g, b;
                    if (v4 < 1) {
                        r = Math.floor(v4 * 128);
                        g = 0;
                        b = 0;
                    } else if (v4 < 2) {
                        r = 128 + Math.floor((v4 - 1) * 127);
                        g = 0;
                        b = 0;
                    } else if (v4 < 3) {
                        r = 255;
                        g = Math.floor((v4 - 2) * 200);
                        b = 0;
                    } else {
                        r = 255;
                        g = 200 + Math.floor((v4 - 3) * 55);
                        b = Math.floor((v4 - 3) * 255);
                    }
                    img.data[idx] = r;
                    img.data[idx + 1] = g;
                    img.data[idx + 2] = b;
                    img.data[idx + 3] = 255;
                }
            }
            ctx.putImageData(img, PAD_L, PAD_T);

            // Filter overlay lines
            const drawFilterLine = (freqHz, label, color) => {
                if (!freqHz || freqHz <= 0 || freqHz >= maxFreqHz) return;
                const x = PAD_L + (freqHz / maxFreqHz) * plotW;
                ctx.strokeStyle = color;
                ctx.lineWidth = 1.5;
                ctx.setLineDash([4, 3]);
                ctx.beginPath();
                ctx.moveTo(x, PAD_T);
                ctx.lineTo(x, PAD_T + plotH);
                ctx.stroke();
                ctx.setLineDash([]);
                ctx.font = "9px monospace";
                ctx.fillStyle = color;
                ctx.textAlign = "left";
                ctx.fillText(label, x + 2, PAD_T + 10);
            };

            if (config) {
                drawFilterLine(
                    config.dtermFilters?.lpf1Hz,
                    `D-LPF1 ${config.dtermFilters?.lpf1Hz}Hz`,
                    "rgba(0,180,200,0.7)",
                );
                drawFilterLine(
                    config.dtermFilters?.lpf2Hz,
                    `D-LPF2 ${config.dtermFilters?.lpf2Hz}Hz`,
                    "rgba(16,140,170,0.7)",
                );
                drawFilterLine(
                    config.dtermFilters?.yawLpfHz,
                    `Yaw LPF ${config.dtermFilters?.yawLpfHz}Hz`,
                    "rgba(80,180,80,0.7)",
                );
                // Dynamic notch range
                const dynMin = config.dynamicNotch?.minHz;
                const dynMax = config.dynamicNotch?.maxHz;
                if (dynMin > 0 && dynMax > dynMin) {
                    drawFilterLine(dynMin, `Dyn notch min`, "rgba(160,100,255,0.5)");
                    drawFilterLine(dynMax, `Dyn notch max`, "rgba(160,100,255,0.5)");
                    // Shaded range
                    const x1 = PAD_L + (dynMin / maxFreqHz) * plotW;
                    const x2 = PAD_L + (dynMax / maxFreqHz) * plotW;
                    ctx.fillStyle = "rgba(160,100,255,0.08)";
                    ctx.fillRect(x1, PAD_T, x2 - x1, plotH);
                }
            }

            // Motor RPM tracking line (cyan) from eRPM data
            const erpmKeys = Object.keys(this._graphFrames[0] || {}).filter((k) => /erpm/i.test(k));
            if (erpmKeys.length > 0 && config) {
                // For each throttle bin, compute average motor frequency
                ctx.strokeStyle = "rgba(0,255,255,0.7)";
                ctx.lineWidth = 2;
                ctx.beginPath();
                let started = false;
                for (let thrBin = 0; thrBin < 100; thrBin++) {
                    let erpmSum = 0,
                        erpmCount = 0;
                    for (const row of this._graphFrames) {
                        const thr = Number(row["rcCommand[3]"] ?? 1000);
                        const rowBin = Math.floor(Math.max(0, Math.min(99.9, (thr - 1000) / 10)));
                        if (Math.abs(rowBin - thrBin) <= 2) {
                            for (const ek of erpmKeys) {
                                const v = Math.abs(Number(row[ek] ?? 0));
                                if (v > 0) {
                                    erpmSum += v;
                                    erpmCount++;
                                }
                            }
                        }
                    }
                    if (erpmCount < 5) continue;
                    const avgErpm = erpmSum / erpmCount;
                    const freqHz = (avgErpm * 100) / 60;
                    if (freqHz < 5 || freqHz > maxFreqHz) continue;
                    const x = PAD_L + (freqHz / maxFreqHz) * plotW;
                    const y = PAD_T + plotH * (1 - thrBin / 100);
                    if (!started) {
                        ctx.moveTo(x, y);
                        started = true;
                    } else ctx.lineTo(x, y);
                }
                ctx.stroke();
                // Label
                if (started) {
                    ctx.font = "9px monospace";
                    ctx.fillStyle = "rgba(0,255,255,0.8)";
                    ctx.fillText("Motor RPM", PAD_L + plotW - 60, PAD_T + plotH - 6);
                }
            }

            // Axis labels
            ctx.font = "10px monospace";
            ctx.fillStyle = "#888";
            ctx.textBaseline = "top";
            ctx.textAlign = "center";
            // X axis: frequency
            for (let f = 0; f <= maxFreqHz; f += 100) {
                const x = PAD_L + (f / maxFreqHz) * plotW;
                ctx.fillText(`${f}`, x, PAD_T + plotH + 4);
            }
            // Y axis: throttle%
            ctx.textBaseline = "middle";
            ctx.textAlign = "right";
            for (let t = 0; t <= 100; t += 20) {
                const y = PAD_T + plotH * (1 - t / 100);
                ctx.fillText(`${t}%`, PAD_L - 4, y);
            }

            // Border
            ctx.strokeStyle = "#333";
            ctx.lineWidth = 1;
            ctx.setLineDash([]);
            ctx.strokeRect(PAD_L, PAD_T, plotW, plotH);
        },

        _renderSpectrogramLegend() {
            const legend = this.$refs.spectrogramLegend;
            if (!legend) return;
            const lctx = legend.getContext("2d");
            const lw = legend.width;
            const lh = legend.height;
            for (let x = 0; x < lw; x++) {
                const v4 = (x / lw) * 4;
                let r, g, b;
                if (v4 < 1) {
                    r = Math.floor(v4 * 128);
                    g = 0;
                    b = 0;
                } else if (v4 < 2) {
                    r = 128 + Math.floor((v4 - 1) * 127);
                    g = 0;
                    b = 0;
                } else if (v4 < 3) {
                    r = 255;
                    g = Math.floor((v4 - 2) * 200);
                    b = 0;
                } else {
                    r = 255;
                    g = 200 + Math.floor((v4 - 3) * 55);
                    b = Math.floor((v4 - 3) * 255);
                }
                lctx.fillStyle = `rgb(${r},${g},${b})`;
                lctx.fillRect(x, 0, 1, lh);
            }
        },

        _renderMotorHeatmap() {
            const canvas = this.$refs.graphMotorHeat;
            if (!canvas || !this.extendedAnalysis?.motorHeatmap) return;
            const ctx = canvas.getContext("2d");
            const W = canvas.width;
            const H = canvas.height;
            const hm = this.extendedAnalysis.motorHeatmap;
            const PAD_L = 64,
                PAD_R = 8,
                PAD_T = 4,
                PAD_B = 22;
            const plotW = W - PAD_L - PAD_R;
            const plotH = H - PAD_T - PAD_B;
            const numMotors = hm.motors.length;
            const numBands = hm.bands;

            ctx.fillStyle = "hsl(0,0%,8%)";
            ctx.fillRect(0, 0, W, H);

            let gMin = Infinity,
                gMax = 0;
            for (const key of hm.motors) {
                for (const v of hm.data[key]) {
                    if (v !== null) {
                        if (v < gMin) gMin = v;
                        if (v > gMax) gMax = v;
                    }
                }
            }
            if (gMax === 0) gMax = 1;
            const range = gMax - gMin || 1;
            const cellW = plotW / numBands;
            const cellH = plotH / numMotors;

            for (let m = 0; m < numMotors; m++) {
                const motorKey = hm.motors[m];
                for (let b = 0; b < numBands; b++) {
                    const v = hm.data[motorKey][b];
                    const cx = PAD_L + b * cellW;
                    const cy = PAD_T + m * cellH;
                    if (v === null) {
                        ctx.fillStyle = "hsl(0,0%,12%)";
                    } else {
                        const norm = (v - gMin) / range;
                        const hue = (1 - norm) * 240;
                        ctx.fillStyle = `hsl(${hue},80%,45%)`;
                    }
                    ctx.fillRect(cx + 1, cy + 1, cellW - 2, cellH - 2);
                    if (v !== null) {
                        ctx.font = "9px monospace";
                        ctx.fillStyle = "#fff";
                        ctx.textAlign = "center";
                        ctx.textBaseline = "middle";
                        ctx.fillText(`${Math.round(v)}`, cx + cellW / 2, cy + cellH / 2);
                    }
                }
                ctx.font = "10px monospace";
                ctx.fillStyle = "#aaa";
                ctx.textAlign = "right";
                ctx.textBaseline = "middle";
                ctx.fillText(
                    motorKey.replace(/[[\]]/g, "").replace("eRPM", "M"),
                    PAD_L - 4,
                    PAD_T + m * cellH + cellH / 2,
                );
            }
            ctx.font = "9px monospace";
            ctx.fillStyle = "#888";
            ctx.textAlign = "center";
            ctx.textBaseline = "top";
            for (let b = 0; b < numBands; b++) {
                ctx.fillText(`${b * 10}%`, PAD_L + b * cellW + cellW / 2, PAD_T + plotH + 4);
            }
        },

        graphZoom(graph, direction) {
            if (direction === 0) {
                this.graphZoomLevels[graph] = 1;
                this.graphPanOffsets[graph] = 0;
            } else if (direction > 0) {
                this.graphZoomLevels[graph] = Math.min(32, this.graphZoomLevels[graph] * 2);
            } else {
                this.graphZoomLevels[graph] = Math.max(1, this.graphZoomLevels[graph] / 2);
            }
            this.renderGraphs();
        },

        applyLogPidsToFC() {
            if (!this.logPidOutput) return;
            const pids = this.logPidOutput.new;
            const store = usePidTuningStore();
            FC.PIDS[0][0] = pids.roll.P;
            FC.PIDS[0][1] = pids.roll.I;
            FC.PIDS[0][2] = pids.roll.D;
            FC.PIDS[1][0] = pids.pitch.P;
            FC.PIDS[1][1] = pids.pitch.I;
            FC.PIDS[1][2] = pids.pitch.D;
            FC.PIDS[2][0] = pids.yaw.P;
            FC.PIDS[2][1] = pids.yaw.I;
            FC.PIDS[2][2] = pids.yaw.D;
            store.needsSave = true;
            mspHelper.sendPidData(() => {
                MSP.send_message(MSPCodes.MSP_EEPROM_WRITE);
            });
        },

        copyLogPids() {
            if (!this.logPidOutput) return;
            const p = this.logPidOutput.new;
            const text = [
                `Roll:  P=${p.roll.P} I=${p.roll.I} D=${p.roll.D}`,
                `Pitch: P=${p.pitch.P} I=${p.pitch.I} D=${p.pitch.D}`,
                `Yaw:   P=${p.yaw.P} I=${p.yaw.I} D=${p.yaw.D}`,
                "",
                `set p_roll = ${p.roll.P}`,
                `set i_roll = ${p.roll.I}`,
                `set d_roll = ${p.roll.D}`,
                `set p_pitch = ${p.pitch.P}`,
                `set i_pitch = ${p.pitch.I}`,
                `set d_pitch = ${p.pitch.D}`,
                `set p_yaw = ${p.yaw.P}`,
                `set i_yaw = ${p.yaw.I}`,
            ].join("\n");
            navigator.clipboard.writeText(text).then(() => {
                this.logPidCopyBtnText = "✔ Copied!";
                setTimeout(() => {
                    this.logPidCopyBtnText = "📋 COPY NEW PIDs";
                }, 2000);
            });
        },

        applyChirpPropDefaults(propInch) {
            const d = chirpDefaultsForProp(propInch);
            this.chirpStartHz = d.startHz;
            this.chirpEndHz = d.endHz;
            // Re-apply current intensity level with prop-appropriate amplitudes
            this.setChirpLevel("pitch", this.chirpPitchLevel);
            this.setChirpLevel("roll", this.chirpRollLevel);
            this.setChirpLevel("yaw", this.chirpYawLevel);
        },

        setChirpLevel(axis, level) {
            const d = chirpDefaultsForProp(this.chirpPropInch);
            const AMPLITUDES = { EASY: d.easy, MEDIUM: d.medium, HARD: d.hard };
            const amp = AMPLITUDES[level];
            if (axis === "pitch") {
                this.chirpPitchLevel = level;
                this.chirpPitch = amp;
            } else if (axis === "roll") {
                this.chirpRollLevel = level;
                this.chirpRoll = amp;
            } else if (axis === "yaw") {
                this.chirpYawLevel = level;
                this.chirpYaw = amp;
            }
        },

        configureFc() {
            if (!CONFIGURATOR.connectionValid) {
                this.chirpConfirmText = "ERROR: Not connected to flight controller.";
                this.chirpConfigured = true;
                return;
            }

            const { chirpStartHz, chirpEndHz, chirpDuration } = this;
            if (
                !Number.isFinite(chirpStartHz) ||
                !Number.isFinite(chirpEndHz) ||
                chirpStartHz >= chirpEndHz ||
                !Number.isFinite(chirpDuration) ||
                chirpDuration < 1 ||
                chirpDuration > 60
            ) {
                this.chirpConfirmText = "ERROR: Invalid chirp sweep settings.";
                this.chirpConfigured = true;
                return;
            }

            this.chirpConfigured = false;

            const startDeciHz = Math.round(chirpStartHz * 10);
            const endDeciHz = Math.round(chirpEndHz * 10);

            const commands = [
                `set chirp_amplitude_pitch = ${this.chirpPitch}`,
                `set chirp_amplitude_roll = ${this.chirpRoll}`,
                `set chirp_amplitude_yaw = ${this.chirpYaw}`,
                `set chirp_frequency_start_deci_hz = ${startDeciHz}`,
                `set chirp_frequency_end_deci_hz = ${endDeciHz}`,
                `set chirp_time_seconds = ${chirpDuration}`,
                `save`,
            ];

            const sendRaw = (str) => {
                const buf = new ArrayBuffer(str.length);
                const view = new Uint8Array(buf);
                for (let i = 0; i < str.length; i++) view[i] = str.codePointAt(i);
                serial.send(buf);
            };

            // Enter CLI mode
            const enterBuf = new ArrayBuffer(1);
            new Uint8Array(enterBuf)[0] = 0x23; // '#'
            serial.send(enterBuf);

            // Wait for CLI to become active before dispatching commands
            const MAX_WAIT_MS = 3000;
            const POLL_INTERVAL_MS = 50;
            let elapsed = 0;
            const poll = setInterval(() => {
                elapsed += POLL_INTERVAL_MS;
                if (CONFIGURATOR.cliActive || CONFIGURATOR.cliValid) {
                    clearInterval(poll);
                    let delay = 0;
                    for (const cmd of commands) {
                        setTimeout(() => sendRaw(`${cmd}\n`), delay);
                        delay += 60;
                    }
                    setTimeout(() => {
                        this.chirpConfirmText = commands.join("\n");
                        this.chirpConfigured = true;
                    }, delay + 100);
                } else if (elapsed >= MAX_WAIT_MS) {
                    clearInterval(poll);
                    this.chirpConfirmText = "ERROR: CLI did not become active. Check connection and try again.";
                }
            }, POLL_INTERVAL_MS);
        },

        /** Switch the active chirp axis and redraw. */
        selectSysIDAxis(axName) {
            this.sysidActiveAxis = axName;
            this.$nextTick(() => this.renderChirpOverlay());
        },

        /** Switch zoom preset and redraw. */
        setSysIDZoom(zoom) {
            this.sysidZoom = zoom;
            this.$nextTick(() => this.renderChirpOverlay());
        },

        /** Render the setpoint vs gyro overlay chart for the active axis. */
        renderChirpOverlay() {
            const canvas = this.$refs.chirpOverlayCanvas;
            if (!canvas || !this.sysidResult) return;
            const axData = this.sysidResult.axes[this.sysidActiveAxis];
            if (!axData || axData.error) return;
            const colors = { roll: "#ff4444", pitch: "#4488ff", yaw: "#44cc66" };
            this._drawChirpOverlay(canvas, axData, colors[this.sysidActiveAxis] || "#ffbb00", this.sysidZoom);
        },

        /** Draw setpoint (axis colour) vs gyro (white) time-domain overlay. */
        _drawChirpOverlay(canvas, axData, axColor, zoom) {
            const { spSeg, gySeg, sampleRate } = axData;
            if (!spSeg || !gySeg || !sampleRate) return;

            const W = canvas.width;
            const H = canvas.height;
            const ctx = canvas.getContext("2d");
            ctx.clearRect(0, 0, W, H);

            const PAD_L = 52,
                PAD_R = 16,
                PAD_T = 18,
                PAD_B = 22;
            const plotW = W - PAD_L - PAD_R;
            const plotH = H - PAD_T - PAD_B;

            const N = spSeg.length;

            // Zoom window: full or middle 50%
            const i0 = zoom === "zoomed" ? Math.floor(N * 0.25) : 0;
            const i1 = zoom === "zoomed" ? Math.floor(N * 0.75) : N - 1;
            const iRange = Math.max(1, i1 - i0);
            const tStart = i0 / sampleRate;
            const tEnd = i1 / sampleRate;

            // Auto-scale amplitude from visible data
            let ampMax = 50;
            for (let i = i0; i <= i1; i++) {
                const v = Math.max(Math.abs(spSeg[i]), Math.abs(gySeg[i]));
                if (v > ampMax) ampMax = v;
            }
            ampMax = Math.ceil(ampMax / 50) * 50;
            const AMP_MIN = -ampMax,
                AMP_MAX = ampMax;

            const xForI = (i) => PAD_L + ((i - i0) / iRange) * plotW;
            const yForA = (a) => PAD_T + plotH - ((clamp(a, AMP_MIN, AMP_MAX) - AMP_MIN) / (AMP_MAX - AMP_MIN)) * plotH;

            // Downsample for performance
            const step = iRange > 1000 ? Math.ceil(iRange / 1000) : 1;

            // Background
            ctx.fillStyle = "#0d1117";
            ctx.fillRect(0, 0, W, H);

            // Zero line (dashed)
            ctx.strokeStyle = "#2a2a2a";
            ctx.lineWidth = 1;
            ctx.setLineDash([4, 4]);
            ctx.beginPath();
            ctx.moveTo(PAD_L, yForA(0));
            ctx.lineTo(W - PAD_R, yForA(0));
            ctx.stroke();
            ctx.setLineDash([]);

            // Horizontal amplitude grid
            ctx.strokeStyle = "#1c1c1c";
            for (let a = -ampMax + 50; a < ampMax; a += 50) {
                if (a === 0) continue;
                ctx.beginPath();
                ctx.moveTo(PAD_L, yForA(a));
                ctx.lineTo(W - PAD_R, yForA(a));
                ctx.stroke();
            }

            // Vertical time grid
            const xTicks = 5;
            for (let t = 0; t <= xTicks; t++) {
                const xi = PAD_L + (t / xTicks) * plotW;
                ctx.beginPath();
                ctx.moveTo(xi, PAD_T);
                ctx.lineTo(xi, H - PAD_B);
                ctx.stroke();
            }

            // Y-axis labels
            ctx.fillStyle = "#555555";
            ctx.font = "10px monospace";
            ctx.textAlign = "right";
            for (let a = -ampMax; a <= ampMax; a += 50) {
                ctx.fillText(String(a), PAD_L - 4, yForA(a) + 3);
            }
            ctx.fillStyle = "#444444";
            ctx.font = "9px monospace";
            ctx.textAlign = "left";
            ctx.fillText("deg/s", 2, PAD_T + 8);

            // X-axis time labels
            ctx.fillStyle = "#555555";
            ctx.font = "10px monospace";
            ctx.textAlign = "center";
            for (let t = 0; t <= xTicks; t++) {
                const tSec = tStart + (t / xTicks) * (tEnd - tStart);
                const xi = PAD_L + (t / xTicks) * plotW;
                ctx.fillText(`${tSec.toFixed(1)}s`, xi, H - PAD_B + 14);
            }

            // Draw gyro trace first (white, behind setpoint)
            ctx.strokeStyle = "rgba(255,255,255,0.65)";
            ctx.lineWidth = 1.2;
            ctx.beginPath();
            let started = false;
            for (let i = i0; i <= i1; i += step) {
                const x = xForI(i);
                const y = yForA(gySeg[i]);
                if (!started) {
                    ctx.moveTo(x, y);
                    started = true;
                } else {
                    ctx.lineTo(x, y);
                }
            }
            ctx.stroke();

            // Draw setpoint trace on top (axis colour)
            ctx.strokeStyle = axColor;
            ctx.lineWidth = 1.4;
            ctx.beginPath();
            started = false;
            for (let i = i0; i <= i1; i += step) {
                const x = xForI(i);
                const y = yForA(spSeg[i]);
                if (!started) {
                    ctx.moveTo(x, y);
                    started = true;
                } else {
                    ctx.lineTo(x, y);
                }
            }
            ctx.stroke();

            // Border
            ctx.strokeStyle = "#2a2a2a";
            ctx.lineWidth = 1;
            ctx.strokeRect(PAD_L, PAD_T, plotW, plotH);
        },

        async applySysIDToFC() {
            if (!this.sysidPids || !CONFIGURATOR.connectionValid) {
                alert("No flight controller connected. Connect to FC before applying PIDs.");
                return;
            }
            const sp = this.sysidPids;
            try {
                await MSP.promise(MSPCodes.MSP_PID);
            } catch (e) {
                console.error("[AeroTune] Failed to read MSP_PID:", e);
                alert("Failed to read PID values from FC. Check connection and try again.");
                return;
            }
            if (FC.PIDS && FC.PIDS.length >= 3) {
                if (sp.roll.P !== "—") FC.PIDS[0][0] = sp.roll.P;
                if (sp.roll.I !== "?") FC.PIDS[0][1] = sp.roll.I;
                if (sp.roll.D !== "—") FC.PIDS[0][2] = sp.roll.D;
                if (sp.pitch.P !== "—") FC.PIDS[1][0] = sp.pitch.P;
                if (sp.pitch.I !== "?") FC.PIDS[1][1] = sp.pitch.I;
                if (sp.pitch.D !== "—") FC.PIDS[1][2] = sp.pitch.D;
                if (sp.yaw.P !== "—") FC.PIDS[2][0] = sp.yaw.P;
                if (sp.yaw.I !== "?") FC.PIDS[2][1] = sp.yaw.I;
            }
            try {
                await MSP.promise(MSPCodes.MSP_PID_ADVANCED);
            } catch (e) {
                console.error("[AeroTune] Failed to read MSP_PID_ADVANCED:", e);
                alert("Failed to read advanced tuning from FC. Check connection and try again.");
                return;
            }
            if (FC.ADVANCED_TUNING) {
                if (sp.roll.DMax !== "—") FC.ADVANCED_TUNING.dMaxRoll = sp.roll.DMax;
                if (sp.pitch.DMax !== "—") FC.ADVANCED_TUNING.dMaxPitch = sp.pitch.DMax;
                if (sp.roll.FF !== "?") FC.ADVANCED_TUNING.feedforwardRoll = sp.roll.FF;
                if (sp.pitch.FF !== "?") FC.ADVANCED_TUNING.feedforwardPitch = sp.pitch.FF;
                if (sp.yaw.FF !== "?") FC.ADVANCED_TUNING.feedforwardYaw = sp.yaw.FF;
            }
            try {
                await MSP.promise(MSPCodes.MSP_SET_PID, mspHelper.crunch(MSPCodes.MSP_SET_PID));
                await MSP.promise(MSPCodes.MSP_SET_PID_ADVANCED, mspHelper.crunch(MSPCodes.MSP_SET_PID_ADVANCED));
            } catch (e) {
                console.error("[AeroTune] Failed to send PID values to FC:", e);
                alert("Failed to send PID values to FC. Check connection and try again.");
                return;
            }
            this.pidTuningStore.markExternalChange();
            const pidTabLink = document.querySelector("li.tab_pid_tuning a");
            if (pidTabLink) {
                pidTabLink.click();
            } else {
                alert("PID Tuning tab not available. Make sure a flight controller is connected.");
            }
        },

        copySysIDValues() {
            if (!this.sysidPids) return;
            const sp = this.sysidPids;
            const text = [
                "# AeroTune SysID PID Values (hover-condition baseline)",
                `Roll   P=${sp.roll.P}  I=${sp.roll.I}  D=${sp.roll.D}  DMax=${sp.roll.DMax}  FF=${sp.roll.FF}`,
                `Pitch  P=${sp.pitch.P}  I=${sp.pitch.I}  D=${sp.pitch.D}  DMax=${sp.pitch.DMax}  FF=${sp.pitch.FF}`,
                `Yaw    P=${sp.yaw.P}  I=${sp.yaw.I}  FF=${sp.yaw.FF}`,
            ].join("\n");
            navigator.clipboard
                .writeText(text)
                .then(() => {
                    this.sysidCopyBtnText = "✔ Copied!";
                    setTimeout(() => {
                        this.sysidCopyBtnText = "📋 COPY ALL VALUES";
                    }, 2000);
                })
                .catch((err) => console.error("[AeroTune] Failed to copy:", err));
        },
    },
};
</script>

<style scoped>
/* ── Chirp overlay chart ─────────────────────────────────────────── */
.at-chirp-overlay-canvas {
    display: block;
    width: 100%;
    max-width: 580px;
    height: auto;
    border: 1px solid #1e2a38;
    border-radius: 3px;
    margin-bottom: 4px;
}
.at-chirp-overlay-legend {
    font-size: 11px;
    color: #666666;
    font-family: monospace;
    margin-bottom: 2px;
}
.at-chirp-overlay-hint {
    font-size: 11px;
    color: #555555;
    font-style: italic;
    font-family: monospace;
    margin-bottom: 10px;
}

/* ── Axis selector tabs ──────────────────────────────────────────── */
.at-sysid-axis-tabs {
    display: flex;
    gap: 6px;
    margin-bottom: 10px;
}
.at-sysid-axis-tab {
    padding: 4px 14px;
    border: 1px solid #333;
    border-radius: 3px;
    background: transparent;
    color: #666;
    font-size: 11px;
    font-weight: 700;
    font-family: monospace;
    letter-spacing: 0.5px;
    cursor: pointer;
    text-transform: uppercase;
    transition:
        background 0.15s,
        color 0.15s,
        border-color 0.15s;
}
.at-sysid-axis-tab:hover {
    color: #aaa;
    border-color: #555;
}
.at-sysid-axis-tab.active {
    color: var(--ax-color, #ffbb00);
    border-color: var(--ax-color, #ffbb00);
    background: color-mix(in srgb, var(--ax-color, #ffbb00) 15%, transparent);
}

/* ── Zoom preset buttons ─────────────────────────────────────────── */
.at-sysid-zoom-row {
    display: flex;
    gap: 6px;
    margin-bottom: 14px;
}
.at-sysid-zoom-btn {
    padding: 3px 12px;
    border: 1px solid #333;
    border-radius: 3px;
    background: transparent;
    color: #666;
    font-size: 11px;
    font-family: monospace;
    cursor: pointer;
    transition:
        background 0.15s,
        color 0.15s,
        border-color 0.15s;
}
.at-sysid-zoom-btn:hover {
    color: #aaa;
    border-color: #555;
}
.at-sysid-zoom-btn.active {
    color: #ffbb00;
    border-color: #ffbb00;
    background: rgba(255, 187, 0, 0.1);
}

/* ── PID output section ──────────────────────────────────────────── */
.at-sysid-pid-output {
    margin-top: 14px;
}
.at-sysid-pid-complete-header {
    color: #00d966;
    font-size: 12px;
    letter-spacing: 0.5px;
    margin-bottom: 10px;
    padding: 8px 12px;
    background: #0d1f0d;
    border-left: 3px solid #00d966;
    border-radius: 2px;
}
.at-sysid-pid-table {
    margin-bottom: 10px;
}
</style>
