<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CNN 인공지능 전체 학습 & 분류 시뮬레이터</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chart.js for Loss Graph -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Font & Custom Styles -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;900&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        input[type=number]::-webkit-inner-spin-button, 
        input[type=number]::-webkit-outer-spin-button { -webkit-appearance: none; margin: 0; }
        input[type=number] { -moz-appearance: textfield; }
        .cell-transition { transition: background-color 0.2s ease, transform 0.1s ease; }
        .tab-active { border-bottom: 4px solid #4F46E5; color: #4F46E5; font-weight: 700; }
    </style>
</head>
<body class="bg-slate-900 text-slate-100 min-h-screen flex flex-col items-center p-4 md:p-8">

    <!-- Header Section -->
    <header class="max-w-6xl w-full text-center mb-6">
        <h1 class="text-3xl md:text-5xl font-black tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-indigo-400 via-purple-400 to-pink-400">
            CNN 라이프사이클 스튜디오
        </h1>
        <p class="text-slate-400 mt-2 text-sm md:text-base">
            합성곱 특징 추출 ➔ 가중치 학습(경사하강법) ➔ 최종 추론(분류)까지 한 번에 체험하기
        </p>
    </header>

    <!-- Navigation Tabs -->
    <nav class="max-w-6xl w-full bg-slate-800 rounded-xl p-1 mb-6 flex justify-around shadow-lg border border-slate-700">
        <button id="tab-btn-1" onclick="switchTab(1)" class="w-1/4 py-3 text-xs md:text-sm font-medium rounded-lg text-slate-400 hover:text-white transition tab-active">
            1단계: 합성곱 추출
        </button>
        <button id="tab-btn-2" onclick="switchTab(2)" class="w-1/4 py-3 text-xs md:text-sm font-medium rounded-lg text-slate-400 hover:text-white transition">
            2단계: 패딩/풀링
        </button>
        <button id="tab-btn-3" onclick="switchTab(3)" class="w-1/4 py-3 text-xs md:text-sm font-medium rounded-lg text-slate-400 hover:text-white transition">
            3단계: 가중치 학습
        </button>
        <button id="tab-btn-4" onclick="switchTab(4)" class="w-1/4 py-3 text-xs md:text-sm font-medium rounded-lg text-slate-400 hover:text-white transition">
            4단계: 파이프라인
        </button>
    </nav>

    <!-- Main Container -->
    <main class="max-w-6xl w-full bg-slate-800/90 backdrop-blur rounded-2xl border border-slate-700 p-6 shadow-2xl min-h-[600px] flex flex-col justify-between">
        
        <!-- SECTION 1: CONVOLUTION FEATURE EXTRACTION -->
        <section id="section-1" class="space-y-6">
            <div class="bg-slate-700/50 p-4 rounded-xl border border-slate-600">
                <h2 class="text-xl font-bold text-indigo-300 flex items-center gap-2">
                    💡 1단계: 돋보기(필터)로 이미지 특징 추출하기
                </h2>
                <p class="text-slate-300 text-sm mt-1">
                    왼쪽 5x5 셀을 클릭해 원하는 모양을 그려보세요. 필터 모양과 이미지 패턴이 <strong class="text-emerald-400">일치하면 큰 양수(+)</strong>, <strong class="text-rose-400">반대면 음수(-)</strong>가 나와 "특징"을 수치로 검출합니다.
                </p>
            </div>

            <!-- Presets -->
            <div class="flex flex-wrap items-center gap-2">
                <span class="text-xs font-semibold uppercase tracking-wider text-slate-400">이미지 프리셋:</span>
                <button onclick="loadImagePreset('v-line')" class="px-3 py-1.5 bg-slate-700 hover:bg-slate-600 rounded-lg text-xs font-semibold">세로선</button>
                <button onclick="loadImagePreset('h-line')" class="px-3 py-1.5 bg-slate-700 hover:bg-slate-600 rounded-lg text-xs font-semibold">가로선</button>
                <button onclick="loadImagePreset('cross')" class="px-3 py-1.5 bg-slate-700 hover:bg-slate-600 rounded-lg text-xs font-semibold">십자가</button>
                <button onclick="loadImagePreset('box')" class="px-3 py-1.5 bg-slate-700 hover:bg-slate-600 rounded-lg text-xs font-semibold">네모</button>
                <span class="text-xs font-semibold uppercase tracking-wider text-slate-400 ml-4">필터:</span>
                <button onclick="setFilterPreset('vertical')" class="px-3 py-1.5 bg-indigo-600 hover:bg-indigo-500 rounded-lg text-xs font-semibold">세로 검출기</button>
                <button onclick="setFilterPreset('horizontal')" class="px-3 py-1.5 bg-indigo-600 hover:bg-indigo-500 rounded-lg text-xs font-semibold">가로 검출기</button>
            </div>

            <!-- Matrices -->
            <div class="flex flex-col md:flex-row items-center justify-center gap-6 py-4">
                <!-- Input Matrix -->
                <div class="flex flex-col items-center">
                    <span class="text-xs font-bold text-slate-400 mb-2">1단계 입력 이미지 (5x5)</span>
                    <div id="input-grid" class="grid grid-cols-5 gap-1.5 bg-slate-900 p-2 rounded-xl border border-slate-700"></div>
                </div>

                <div class="text-2xl font-bold text-slate-500">✕</div>

                <!-- Filter Matrix -->
                <div class="flex flex-col items-center">
                    <span class="text-xs font-bold text-indigo-400 mb-2">가중치 필터 W (3x3)</span>
                    <div id="filter-grid" class="grid grid-cols-3 gap-1.5 bg-slate-900 p-2 rounded-xl border border-indigo-900"></div>
                </div>

                <div class="text-2xl font-bold text-slate-500">=</div>

                <!-- Output Feature Map -->
                <div class="flex flex-col items-center">
                    <span class="text-xs font-bold text-emerald-400 mb-2">특징 맵 Feature Map (3x3)</span>
                    <div id="output-grid" class="grid grid-cols-3 gap-1.5 bg-slate-900 p-2 rounded-xl border border-emerald-900"></div>
                </div>
            </div>

            <!-- Calculation Detail Box -->
            <div class="bg-slate-950 p-4 rounded-xl border border-slate-800 text-center font-mono text-sm min-h-[90px] flex flex-col justify-center">
                <div id="step-math-display" class="text-slate-300">
                    '다음 단계' 또는 '자동 연산' 버튼을 누르면 원소별 곱셈과 덧셈 과정을 시각화합니다.
                </div>
            </div>

            <!-- Step Controls -->
            <div class="flex justify-center gap-3">
                <button onclick="resetConvStep()" class="px-5 py-2.5 bg-slate-700 hover:bg-slate-600 rounded-xl font-semibold text-sm">초기화</button>
                <button onclick="nextConvStep()" class="px-5 py-2.5 bg-indigo-600 hover:bg-indigo-500 rounded-xl font-semibold text-sm">다음 단계 (1칸 이동)</button>
                <button id="play-conv-btn" onclick="toggleConvPlay()" class="px-5 py-2.5 bg-emerald-600 hover:bg-emerald-500 rounded-xl font-semibold text-sm">전체 자동 실행</button>
            </div>
        </section>

        <!-- SECTION 2: PADDING, STRIDE, POOLING -->
        <section id="section-2" class="space-y-6 hidden">
            <div class="bg-slate-700/50 p-4 rounded-xl border border-slate-600">
                <h2 class="text-xl font-bold text-amber-300 flex items-center gap-2">
                    🛠️ 2단계: 1단계에서 얻은 특징 맵 요약하기 (패딩 & 풀링)
                </h2>
                <p class="text-slate-300 text-sm mt-1">
                    <strong class="text-emerald-400">1단계의 합성곱 연산 결과(특징 맵)</strong>를 가져와서 핵심만 뽑아내거나(Pooling), 외곽 테두리를 보존(Padding)합니다.
                </p>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <!-- Padding & Stride -->
                <div class="bg-slate-900 p-5 rounded-xl border border-slate-700 space-y-4">
                    <h3 class="text-sm font-bold text-amber-400 uppercase tracking-wide">패딩(Padding) & 스트라이드(Stride)</h3>
                    <div class="flex flex-col gap-2 bg-slate-800 p-3 rounded-lg">
                        <label class="flex items-center gap-2 text-sm text-slate-300 cursor-pointer">
                            <input type="checkbox" id="pad-toggle" onchange="updatePadStride()" class="w-4 h-4 accent-amber-500">
                            <strong>패딩 (Padding=1):</strong> 이미지 테두리에 0을 둘러 크기 축소 방지
                        </label>
                        <label class="flex items-center gap-2 text-sm text-slate-300 cursor-pointer">
                            <input type="checkbox" id="stride-toggle" onchange="updatePadStride()" class="w-4 h-4 accent-amber-500">
                            <strong>스트라이드 (Stride=2):</strong> 필터를 한 번에 두 칸씩 이동
                        </label>
                    </div>
                    
                    <div class="flex items-center justify-center gap-4 py-2">
                        <div class="flex flex-col items-center">
                            <span class="text-xs text-slate-400 mb-1">1단계 입력 이미지 (5x5)</span>
                            <div id="pad-input-grid" class="grid gap-1 bg-slate-950 p-1.5 rounded-lg border border-slate-700 transition-all"></div>
                        </div>
                        <div class="flex flex-col items-center text-slate-500 text-xs">
                            <span>필터(3x3)</span>
                            <span class="text-xl font-bold">➔</span>
                        </div>
                        <div class="flex flex-col items-center">
                            <span class="text-xs text-slate-400 mb-1">변환된 특징 맵</span>
                            <div id="pad-output-grid" class="grid gap-1 bg-slate-950 p-1.5 rounded-lg border border-amber-900 transition-all"></div>
                        </div>
                    </div>
                    <div class="bg-slate-950 p-3 rounded-lg border border-slate-800 text-xs font-mono text-center text-amber-200" id="pad-stride-formula">
                        출력 크기 = (5 - 3 + 2×0) / 1 + 1 = 3x3
                    </div>
                </div>

                <!-- Max Pooling -->
                <div class="bg-slate-900 p-5 rounded-xl border border-slate-700 space-y-4 flex flex-col justify-between">
                    <div>
                        <h3 class="text-sm font-bold text-cyan-400 uppercase tracking-wide">맥스 풀링 (Max Pooling 2x2)</h3>
                        <p class="text-xs text-slate-400 mt-1">1단계에서 생성된 특징 맵 중에서 가장 수치가 높았던(특징이 강한) 값만 선택해 2x2로 줄입니다.</p>
                    </div>
                    
                    <div class="flex items-center justify-center gap-4 my-auto">
                        <div class="flex flex-col items-center">
                            <span class="text-xs text-indigo-400 mb-1">1단계 결과 특징 맵 (3x3 $\rightarrow$ 4x4 확장)</span>
                            <div id="pool-input-grid" class="grid grid-cols-4 gap-1 bg-slate-950 p-1.5 rounded-lg border border-slate-700"></div>
                        </div>
                        <span class="text-slate-500 text-xl font-bold">➔</span>
                        <div class="flex flex-col items-center">
                            <span class="text-xs text-cyan-400 mb-1">압축된 특징 (2x2)</span>
                            <div id="pool-output-grid" class="grid grid-cols-2 gap-1 bg-slate-950 p-1.5 rounded-lg border border-cyan-900"></div>
                        </div>
                    </div>

                    <div class="flex justify-center gap-2 mt-4">
                        <button onclick="resetPool()" class="px-4 py-2 bg-slate-700 hover:bg-slate-600 rounded-lg text-sm font-bold">초기화</button>
                        <button onclick="nextPoolStep()" class="px-4 py-2 bg-cyan-600 hover:bg-cyan-500 rounded-lg text-sm font-bold shadow-lg shadow-cyan-900/50">다음 맥스풀링 영역 찾기</button>
                    </div>
                </div>
            </div>
        </section>


        <!-- SECTION 3: WEIGHT TRAINING (BACKPROPAGATION & GRADIENT DESCENT) -->
        <section id="section-3" class="space-y-6 hidden">
            <div class="bg-slate-700/50 p-4 rounded-xl border border-slate-600">
                <h2 class="text-xl font-bold text-purple-300 flex items-center gap-2">
                    🎯 3단계: 내 그림에 맞춰 가중치(필터) 수정하기 (학습)
                </h2>
                <p class="text-slate-300 text-sm mt-1">
                    처음에는 아무 특징도 못 찾는 엉터리 가중치 필터에서 시작합니다. <strong class="text-pink-400">1단계에서 내가 만든 그림</strong>을 검출할 수 있도록 인공지능이 필터 숫자를 스스로 수정(경사하강법)하는 과정을 관찰해 보세요!
                </p>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <!-- Training Controls & Matrix -->
                <div class="bg-slate-900 p-5 rounded-xl border border-slate-700 space-y-4">
                    <h3 class="text-sm font-bold text-indigo-400 uppercase tracking-wide">학습 진행 상황</h3>
                    
                    <!-- Target Objective -->
                    <div class="flex flex-col gap-1 text-sm bg-slate-800 p-3 rounded-lg border border-indigo-500/30">
                        <div class="flex justify-between">
                            <span class="text-slate-400">학습 목표 대상:</span>
                            <span class="font-bold text-emerald-400">1단계 사용자가 그린 그림</span>
                        </div>
                        <span class="text-xs text-slate-400 mt-1">💡 "이 패턴을 가장 강하게 감지하도록 3x3 필터를 깎아나갑니다."</span>
                    </div>

                    <!-- Current Filter W Matrix -->
                    <div class="flex flex-col items-center">
                        <span class="text-xs text-slate-400 mb-2">실시간 수정 중인 가중치 필터 W (3x3)</span>
                        <div id="train-filter-grid" class="grid grid-cols-3 gap-2 bg-slate-950 p-2 rounded-xl border border-purple-500"></div>
                    </div>

                    <!-- Learning Rate & Parameters -->
                    <div class="space-y-2">
                        <div class="flex justify-between text-xs font-semibold text-slate-300">
                            <span>학습률 (Learning Rate, $\eta$): <span id="lr-val-text" class="text-indigo-400 font-bold">0.1</span></span>
                        </div>
                        <input id="lr-slider" type="range" min="0.01" max="0.5" step="0.01" value="0.1" oninput="updateLRText(this.value)" class="w-full accent-indigo-500 cursor-pointer">
                        <p class="text-xs text-slate-500">학습률을 바꾸면 가중치 숫자가 갱신되는 속도가 변합니다.</p>
                    </div>

                    <!-- Action Buttons -->
                    <div class="flex gap-2 pt-2">
                        <button onclick="trainStep()" class="w-1/2 py-2.5 bg-purple-600 hover:bg-purple-500 rounded-lg text-sm font-bold">1회 학습 (1 Epoch)</button>
                        <button id="auto-train-btn" onclick="toggleAutoTrain()" class="w-1/2 py-2.5 bg-pink-600 hover:bg-pink-500 rounded-lg text-sm font-bold">연속 학습 실행</button>
                        <button onclick="resetTraining()" class="px-3 py-2.5 bg-slate-700 hover:bg-slate-600 rounded-lg text-sm font-bold">초기화</button>
                    </div>
                </div>

                <!-- Loss Graph & Math Explanation -->
                <div class="bg-slate-900 p-5 rounded-xl border border-slate-700 flex flex-col justify-between">
                    <div>
                        <h3 class="text-sm font-bold text-pink-400 uppercase tracking-wide mb-2">오차(Loss) 감소 그래프</h3>
                        <div class="h-48 w-full bg-slate-950 p-2 rounded-lg border border-slate-800">
                            <canvas id="lossChart"></canvas>
                        </div>
                    </div>

                    <!-- Formula Box -->
                    <div class="bg-slate-950 p-3 rounded-lg border border-slate-800 mt-4 text-xs font-mono space-y-1">
                        <div class="text-indigo-300 font-bold">가중치 수정 원리 (역전파 & 경사하강법):</div>
                        <div class="text-slate-300"> $W_{새로운값} = W_{기존값} - (\text{학습률} \times \text{기울기})$</div>
                        <div class="text-emerald-400 text-[11px] mt-1">✓ 오차가 0에 도달하면, 완료된 가중치 필터가 4단계 전체 파이프라인에 반영됩니다!</div>
                    </div>
                </div>
            </div>
        </section>


        <!-- SECTION 4: FULL CNN PIPELINE & PREDICTION -->
        <section id="section-4" class="space-y-6 hidden">
            <div class="bg-slate-700/50 p-4 rounded-xl border border-slate-600">
                <h2 class="text-xl font-bold text-emerald-300 flex items-center gap-2">
                    🚀 4단계: 1~3단계를 거친 신경망으로 최종 분류 테스트하기
                </h2>
                <p class="text-slate-300 text-sm mt-1">
                    1단계의 이미지, 2단계의 압축, 3단계에서 정교하게 학습된 <strong class="text-purple-300">가중치 필터</strong>가 어떻게 뇌 신경망 역할을 하여 픽셀을 인식하는지 종합 확인합니다.
                </p>
            </div>

            <!-- Drawing Board & Pipeline Grid -->
            <div class="grid grid-cols-1 lg:grid-cols-12 gap-6 items-center">
                
                <!-- 1. Canvas Input (3 cols) -->
                <div class="lg:col-span-4 bg-slate-900 p-4 rounded-xl border border-slate-700 flex flex-col items-center">
                    <span class="text-xs font-bold text-slate-300 mb-2">입력 이미지 (1단계와 연동)</span>
                    <div id="draw-grid" class="grid grid-cols-5 gap-1.5 bg-slate-950 p-2 rounded-xl border border-slate-800 cursor-pointer"></div>
                    <div class="flex gap-2 mt-3 w-full">
                        <button onclick="clearDrawGrid()" class="w-full py-1.5 bg-slate-700 hover:bg-slate-600 rounded text-xs font-bold">지우기</button>
                    </div>
                </div>

                <!-- Pipeline Arrows & Stages (5 cols) -->
                <div class="lg:col-span-5 bg-slate-950 p-4 rounded-xl border border-slate-800 space-y-4">
                    <div class="text-xs font-semibold text-slate-400 text-center uppercase tracking-wider">실시간 연동 파이프라인</div>
                    
                    <div class="flex items-center justify-between gap-1 text-center text-xs">
                        <!-- Stage A: Input -->
                        <div class="flex flex-col items-center">
                            <span class="text-slate-400 font-bold mb-1">입력</span>
                            <span class="p-2 bg-slate-800 rounded border border-slate-700">5x5</span>
                        </div>
                        <span class="text-indigo-400 font-bold">➔</span>
                        <!-- Stage B: Conv + ReLU -->
                        <div class="flex flex-col items-center">
                            <span class="text-indigo-300 font-bold mb-1">3단계 필터 적용</span>
                            <span class="p-2 bg-indigo-950 text-indigo-200 rounded border border-indigo-700">3x3</span>
                        </div>
                        <span class="text-emerald-400 font-bold">➔</span>
                        <!-- Stage C: Max Pool -->
                        <div class="flex flex-col items-center">
                            <span class="text-emerald-300 font-bold mb-1">Max Pool</span>
                            <span class="p-2 bg-emerald-950 text-emerald-200 rounded border border-emerald-700">2x2</span>
                        </div>
                        <span class="text-purple-400 font-bold">➔</span>
                        <!-- Stage D: Flatten -->
                        <div class="flex flex-col items-center">
                            <span class="text-purple-300 font-bold mb-1">Softmax</span>
                            <span class="p-2 bg-purple-950 text-purple-200 rounded border border-purple-700">분류</span>
                        </div>
                    </div>

                    <div class="bg-slate-900 p-3 rounded-lg border border-slate-800 text-xs text-slate-300 space-y-1 font-mono">
                        <div class="text-indigo-300">현재 활성화된 학습 필터 W:</div>
                        <div id="pipeline-filter-preview" class="text-[11px] text-slate-400">3단계에서 학습된 가중치 숫자가 여기에 실시간으로 반영되어 연산에 사용됩니다.</div>
                    </div>
                </div>

                <!-- 3. Prediction Output (3 cols) -->
                <div class="lg:col-span-3 bg-slate-900 p-4 rounded-xl border border-slate-700 flex flex-col justify-between h-full">
                    <span class="text-xs font-bold text-pink-400 mb-2 text-center">최종 분류 확신도 (Softmax)</span>
                    
                    <div id="prediction-results" class="space-y-3 my-auto">
                        <!-- JS Dynamic Insertion -->
                    </div>

                    <div class="text-[11px] text-slate-500 text-center mt-3">
                        3단계에서 가중치를 충분히 학습시켰다면 그려진 패턴의 확신도가 90% 이상 올라갑니다!
                    </div>
                </div>
            </div>
        </section>


        <!-- SECTION 3: WEIGHT TRAINING (BACKPROPAGATION & GRADIENT DESCENT) -->
        <section id="section-3" class="space-y-6 hidden">
            <div class="bg-slate-700/50 p-4 rounded-xl border border-slate-600">
                <h2 class="text-xl font-bold text-purple-300 flex items-center gap-2">
                    🎯 가중치(Weight) 수정 과정 체험 (학습/Train)
                </h2>
                <p class="text-slate-300 text-sm mt-1">
                    인공지능은 처음부터 똑똑하지 않습니다. 정답과의 <strong class="text-pink-400">오차(Loss)</strong>를 계산한 뒤, 경사하강법(Gradient Descent) 공식을 통해 가중치 필터 $W$를 조금씩 수정하여 목표 특징(예: 세로선)을 추출하도록 학습합니다.
                </p>
            </div>

            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <!-- Training Controls & Matrix -->
                <div class="bg-slate-900 p-5 rounded-xl border border-slate-700 space-y-4">
                    <h3 class="text-sm font-bold text-indigo-400 uppercase tracking-wide">학습 설정 및 시뮬레이션</h3>
                    
                    <!-- Target Objective -->
                    <div class="flex items-center justify-between text-sm bg-slate-800 p-3 rounded-lg">
                        <span>학습 목표:</span>
                        <span class="font-bold text-emerald-400">세로선(Vertical Edge)을 잘 찾아내는 필터 만들기</span>
                    </div>

                    <!-- Current Filter W Matrix -->
                    <div class="flex flex-col items-center">
                        <span class="text-xs text-slate-400 mb-2">현재 필터 가중치 $W$ (실시간 수정됨)</span>
                        <div id="train-filter-grid" class="grid grid-cols-3 gap-2 bg-slate-950 p-2 rounded-xl border border-purple-500"></div>
                    </div>

                    <!-- Learning Rate & Parameters -->
                    <div class="space-y-2">
                        <div class="flex justify-between text-xs font-semibold text-slate-300">
                            <span>학습률 (Learning Rate, $\eta$): <span id="lr-val-text" class="text-indigo-400 font-bold">0.1</span></span>
                        </div>
                        <input id="lr-slider" type="range" min="0.01" max="0.5" step="0.01" value="0.1" oninput="updateLRText(this.value)" class="w-full accent-indigo-500 cursor-pointer">
                        <p class="text-xs text-slate-500">학습률이 너무 크면 가중치가 발산하고, 너무 작으면 학습이 너무 느립니다.</p>
                    </div>

                    <!-- Action Buttons -->
                    <div class="flex gap-2 pt-2">
                        <button onclick="trainStep()" class="w-1/2 py-2.5 bg-purple-600 hover:bg-purple-500 rounded-lg text-sm font-bold">1회 학습 (1 Epoch)</button>
                        <button id="auto-train-btn" onclick="toggleAutoTrain()" class="w-1/2 py-2.5 bg-pink-600 hover:bg-pink-500 rounded-lg text-sm font-bold">연속 학습 (10 Epoch)</button>
                        <button onclick="resetTraining()" class="px-3 py-2.5 bg-slate-700 hover:bg-slate-600 rounded-lg text-sm font-bold">초기화</button>
                    </div>
                </div>

                <!-- Loss Graph & Math Explanation -->
                <div class="bg-slate-900 p-5 rounded-xl border border-slate-700 flex flex-col justify-between">
                    <div>
                        <h3 class="text-sm font-bold text-pink-400 uppercase tracking-wide mb-2">손실(Loss) 변화 그래프</h3>
                        <div class="h-48 w-full bg-slate-950 p-2 rounded-lg border border-slate-800">
                            <canvas id="lossChart"></canvas>
                        </div>
                    </div>

                    <!-- Formula Box -->
                    <div class="bg-slate-950 p-3 rounded-lg border border-slate-800 mt-4 text-xs font-mono space-y-1">
                        <div class="text-indigo-300 font-bold">가중치 수정 공식 (경사하강법):</div>
                        <div class="text-slate-300"> $W_{new} = W_{old} - (\eta \times \text{Gradient})$</div>
                        <div class="text-slate-400 text-[11px] mt-1">오차(Loss)가 0에 가까워질수록 필터가 세로 특징을 추출할 수 있는 상태로 완성됩니다.</div>
                    </div>
                </div>
            </div>
        </section>


        <!-- SECTION 4: FULL CNN PIPELINE & PREDICTION -->
        <section id="section-4" class="space-y-6 hidden">
            <div class="bg-slate-700/50 p-4 rounded-xl border border-slate-600">
                <h2 class="text-xl font-bold text-emerald-300 flex items-center gap-2">
                    🚀 완성된 CNN 파이프라인 & 분류 추론
                </h2>
                <p class="text-slate-300 text-sm mt-1">
                    특징 추출부터 마지막 클래스 확률 계산(Softmax)까지, 전체 CNN이 동작하는 과정을 시각적으로 확인합니다.
                </p>
            </div>

            <!-- Drawing Board & Pipeline Grid -->
            <div class="grid grid-cols-1 lg:grid-cols-12 gap-6 items-center">
                
                <!-- 1. Canvas Input (3 cols) -->
                <div class="lg:col-span-3 bg-slate-900 p-4 rounded-xl border border-slate-700 flex flex-col items-center">
                    <span class="text-xs font-bold text-slate-300 mb-2">입력 패드 (그려서 입력)</span>
                    <div id="draw-grid" class="grid grid-cols-5 gap-1.5 bg-slate-950 p-2 rounded-xl border border-slate-800 cursor-pointer"></div>
                    <div class="flex gap-2 mt-3 w-full">
                        <button onclick="clearDrawGrid()" class="w-full py-1.5 bg-slate-700 hover:bg-slate-600 rounded text-xs font-bold">지우기</button>
                    </div>
                </div>

                <!-- Pipeline Arrows & Stages (6 cols) -->
                <div class="lg:col-span-6 bg-slate-950 p-4 rounded-xl border border-slate-800 space-y-4">
                    <div class="text-xs font-semibold text-slate-400 text-center uppercase tracking-wider">CNN 내부 데이터 흐름</div>
                    
                    <div class="flex items-center justify-between gap-1 text-center text-xs">
                        <!-- Stage A: Input -->
                        <div class="flex flex-col items-center">
                            <span class="text-slate-400 font-bold mb-1">입력</span>
                            <span class="p-2 bg-slate-800 rounded border border-slate-700">5x5</span>
                        </div>
                        <span class="text-indigo-400 font-bold">➔</span>
                        <!-- Stage B: Conv + ReLU -->
                        <div class="flex flex-col items-center">
                            <span class="text-indigo-300 font-bold mb-1">Conv+ReLU</span>
                            <span class="p-2 bg-indigo-950 text-indigo-200 rounded border border-indigo-700">3x3</span>
                        </div>
                        <span class="text-emerald-400 font-bold">➔</span>
                        <!-- Stage C: Max Pool -->
                        <div class="flex flex-col items-center">
                            <span class="text-emerald-300 font-bold mb-1">Max Pooling</span>
                            <span class="p-2 bg-emerald-950 text-emerald-200 rounded border border-emerald-700">2x2</span>
                        </div>
                        <span class="text-purple-400 font-bold">➔</span>
                        <!-- Stage D: Flatten -->
                        <div class="flex flex-col items-center">
                            <span class="text-purple-300 font-bold mb-1">Flatten</span>
                            <span class="p-2 bg-purple-950 text-purple-200 rounded border border-purple-700">4x1</span>
                        </div>
                    </div>

                    <div class="bg-slate-900 p-3 rounded-lg border border-slate-800 text-xs text-slate-300 space-y-1">
                        <div>• <strong>Conv+ReLU:</strong> 음수 특징은 0으로 바꾸어 중요한 특징 강조</div>
                        <div>• <strong>Max Pooling:</strong> 가장 강한 특징만 남겨 데이터 크기 축소</div>
                        <div>• <strong>Flatten:</strong> 2D 특징 맵을 1D 배열로 펼쳐 뇌 신경망에 연결</div>
                    </div>
                </div>

                <!-- 3. Prediction Output (3 cols) -->
                <div class="lg:col-span-3 bg-slate-900 p-4 rounded-xl border border-slate-700 flex flex-col justify-between h-full">
                    <span class="text-xs font-bold text-pink-400 mb-2 text-center">최종 분류 결과 (Softmax)</span>
                    
                    <div id="prediction-results" class="space-y-3 my-auto">
                        <!-- JS Dynamic Insertion -->
                    </div>

                    <div class="text-[11px] text-slate-500 text-center mt-3">
                        입력 그리드 모양에 따라 인공지능이 클래스 확률을 실시간 계산합니다.
                    </div>
                </div>
            </div>
        </section>

    </main>

    <!-- Footer -->
    <footer class="mt-6 text-xs text-slate-500 text-center">
        고등학교 인공지능 알고리즘 발표 및 실습용 시뮬레이터 | CNN Full Lifecycle Visualizer
    </footer>

    <!-- Main Logic Script -->
    <script>
        // GLOBAL STATES
        let currentTab = 1;

        // SHARED STATE ACROSS STAGES
        let inputData = [
            [0,0,1,0,0],
            [0,0,1,0,0],
            [0,0,1,0,0],
            [0,0,1,0,0],
            [0,0,1,0,0]
        ];
        let filterData = [
            [1, 0, -1],
            [1, 0, -1],
            [1, 0, -1]
        ];
        let outputData = Array(3).fill().map(() => Array(3).fill(null));
        let convStep = -1;
        let convPlayInterval = null;

        // SECTION 3 WEIGHT TRAINING STATE
        let trainFilter = [
            [0.1, -0.2, 0.0],
            [0.3, 0.1, -0.1],
            [-0.1, 0.2, 0.0]
        ];
        let learningRate = 0.1;
        let epochCount = 0;
        let lossHistory = [];
        let lossChart = null;
        let autoTrainInterval = null;

        // SECTION 2 STATE (POOLING & HYPERPARAMS)
        let poolInputData = Array(4).fill().map(() => Array(4).fill(0));
        let poolOutputData = [[null, null], [null, null]];
        let poolStep = -1;

        // TAB SWITCHING & CONTINUOUS DATA SYNC
        function switchTab(tabNum) {
            currentTab = tabNum;
            for(let i=1; i<=4; i++) {
                const sec = document.getElementById(`section-${i}`);
                const btn = document.getElementById(`tab-btn-${i}`);
                if (i === tabNum) {
                    sec.classList.remove('hidden');
                    btn.classList.add('tab-active');
                } else {
                    sec.classList.add('hidden');
                    btn.classList.remove('tab-active');
                }
            }
            
            // Sync Stage 1 -> Stage 2 & 3 & 4
            syncDataPipeline();

            if (tabNum === 2) {
                updatePadStride();
                renderPool();
            }
            if (tabNum === 3) {
                renderTrainFilter();
                if (!lossChart) initLossChart();
            }
            if (tabNum === 4) {
                renderDrawGrid();
                updatePipelinePrediction();
            }
        }

        // CONTINUOUS DATA PIPELINE SYNCHRONIZATION
        function syncDataPipeline() {
            // Compute full 3x3 output feature map from Stage 1 input & filter
            for (let r = 0; r < 3; r++) {
                for (let c = 0; c < 3; c++) {
                    let sum = 0;
                    for (let fr = 0; fr < 3; fr++) {
                        for (let fc = 0; fc < 3; fc++) {
                            sum += inputData[r + fr][c + fc] * filterData[fr][fc];
                        }
                    }
                    outputData[r][c] = sum;
                }
            }

            // Sync to Stage 2 Pooling (Expand 3x3 output to 4x4 for smooth pooling demo)
            for(let r=0; r<4; r++) {
                for(let c=0; c<4; c++) {
                    let rIdx = Math.min(2, r);
                    let cIdx = Math.min(2, c);
                    poolInputData[r][c] = Math.max(0, outputData[rIdx][cIdx]); // Apply ReLU (no negative values)
                }
            }
        }

        // ================= SECTION 1 LOGIC =================
        function renderSection1() {
            renderGrid('input-grid', inputData, true, toggleInputCell);
            renderGrid('filter-grid', filterData, true, (r,c) => {
                let current = filterData[r][c];
                filterData[r][c] = current === 1 ? 0 : (current === 0 ? -1 : 1);
                resetConvStep();
                renderSection1();
                syncDataPipeline();
            });
            renderGrid('output-grid', outputData, false);
        }

        function renderGrid(containerId, data, clickable = false, onClickCallback = null) {
            const container = document.getElementById(containerId);
            container.innerHTML = '';
            const rows = data.length;
            const cols = data[0].length;

            for(let r=0; r<rows; r++) {
                for(let c=0; c<cols; c++) {
                    const cell = document.createElement('div');
                    const val = data[r][c];
                    
                    let bgColor = 'bg-slate-800 text-slate-200';
                    if (val > 0 && containerId === 'input-grid') bgColor = 'bg-indigo-600 text-white font-bold';
                    if (val === 0 && containerId === 'input-grid') bgColor = 'bg-slate-950 text-slate-500';

                    cell.className = `cell-transition w-10 h-10 md:w-12 md:h-12 flex items-center justify-center text-xs md:text-sm rounded-lg font-mono cursor-pointer border border-slate-700/50 ${bgColor}`;
                    cell.id = `${containerId}-${r}-${c}`;
                    cell.innerText = val !== null ? (typeof val === 'number' ? (Number.isInteger(val) ? val : val.toFixed(1)) : '') : '';

                    if(clickable && onClickCallback) {
                        cell.onclick = () => onClickCallback(r, c);
                    }
                    container.appendChild(cell);
                }
            }
        }

        function toggleInputCell(r, c) {
            inputData[r][c] = inputData[r][c] === 1 ? 0 : 1;
            resetConvStep();
            renderSection1();
            syncDataPipeline();
        }

        function loadImagePreset(type) {
            if (type === 'v-line') {
                inputData = [[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0],[0,0,1,0,0]];
            } else if (type === 'h-line') {
                inputData = [[0,0,0,0,0],[0,0,0,0,0],[1,1,1,1,1],[0,0,0,0,0],[0,0,0,0,0]];
            } else if (type === 'cross') {
                inputData = [[0,0,1,0,0],[0,0,1,0,0],[1,1,1,1,1],[0,0,1,0,0],[0,0,1,0,0]];
            } else if (type === 'box') {
                inputData = [[1,1,1,1,1],[1,0,0,0,1],[1,0,0,0,1],[1,0,0,0,1],[1,1,1,1,1]];
            }
            resetConvStep();
            renderSection1();
            syncDataPipeline();
        }

        function setFilterPreset(type) {
            if (type === 'vertical') {
                filterData = [[1,0,-1],[1,0,-1],[1,0,-1]];
            } else if (type === 'horizontal') {
                filterData = [[1,1,1],[0,0,0],[-1,-1,-1]];
            }
            resetConvStep();
            renderSection1();
            syncDataPipeline();
        }

        function resetConvStep() {
            stopConvPlay();
            convStep = -1;
            outputData = Array(3).fill().map(() => Array(3).fill(null));
            document.getElementById('step-math-display').innerHTML = "'다음 단계' 또는 '자동 연산' 버튼을 누르면 원소별 곱셈과 덧셈 과정을 시각화합니다.";
            renderSection1();
            clearHighlights();
        }

        function clearHighlights() {
            document.querySelectorAll('#input-grid > div, #filter-grid > div, #output-grid > div').forEach(el => {
                el.classList.remove('ring-2', 'ring-amber-400', 'bg-indigo-900', 'bg-emerald-900', 'scale-105');
            });
        }

        function nextConvStep() {
            if (convStep >= 8) {
                stopConvPlay();
                document.getElementById('step-math-display').innerHTML = "<span class='text-emerald-400 font-bold'>전체 합성곱 연산 완료! 입력 패턴과 필터 형태가 매칭되는 위치에서 높은 양수가 검출되었습니다.</span>";
                return;
            }

            convStep++;
            clearHighlights();

            const outR = Math.floor(convStep / 3);
            const outC = convStep % 3;
            let sum = 0;
            let calcParts = [];

            for (let fr = 0; fr < 3; fr++) {
                for (let fc = 0; fc < 3; fc++) {
                    const inR = outR + fr;
                    const inC = outC + fc;
                    const inVal = inputData[inR][inC];
                    const filVal = filterData[fr][fc];
                    const prod = inVal * filVal;
                    sum += prod;

                    const inCell = document.getElementById(`input-grid-${inR}-${inC}`);
                    if (inCell) inCell.classList.add('ring-2', 'ring-amber-400', 'bg-indigo-900');

                    const filCell = document.getElementById(`filter-grid-${fr}-${fc}`);
                    if (filCell) filCell.classList.add('ring-2', 'ring-amber-400');

                    calcParts.push(`(${inVal}×${filVal})`);
                }
            }

            outputData[outR][outC] = sum;
            const outCell = document.getElementById(`output-grid-${outR}-${outC}`);
            if (outCell) {
                outCell.innerText = sum;
                outCell.classList.add('ring-2', 'ring-emerald-400', 'bg-emerald-800', 'scale-105');
            }

            let meaningText = sum > 0 ? "👉 패턴 감지됨 (양수)" : (sum < 0 ? "👉 반대 패턴 (음수)" : "👉 특징 없음 (0)");

            document.getElementById('step-math-display').innerHTML = 
                `<div class='text-xs text-slate-400 mb-1'>위치 (${outR+1}행, ${outC+1}열) 연산:</div>` +
                `<div class='text-slate-200'>${calcParts.join(' + ')}</div>` +
                `<div class='text-lg font-bold text-amber-400 mt-1'>= ${sum} <span class='text-xs text-indigo-300 ml-2'>${meaningText}</span></div>`;
        }

        function toggleConvPlay() {
            if (convPlayInterval) {
                stopConvPlay();
            } else {
                if (convStep >= 8) resetConvStep();
                document.getElementById('play-conv-btn').innerText = "일시 정지";
                nextConvStep();
                convPlayInterval = setInterval(() => {
                    nextConvStep();
                    if (convStep >= 8) stopConvPlay();
                }, 1200);
            }
        }

        function stopConvPlay() {
            if (convPlayInterval) {
                clearInterval(convPlayInterval);
                convPlayInterval = null;
            }
            document.getElementById('play-conv-btn').innerText = "전체 자동 실행";
        }

        // ================= SECTION 2 LOGIC =================
        function updatePadStride() {
            const isPad = document.getElementById('pad-toggle').checked;
            const isStride2 = document.getElementById('stride-toggle').checked;
            
            const inSize = isPad ? 7 : 5;
            const stride = isStride2 ? 2 : 1;
            const filterSize = 3;
            
            const padding = isPad ? 1 : 0;
            const outSize = Math.floor((5 - filterSize + 2 * padding) / stride) + 1;

            const padFormula = document.getElementById('pad-stride-formula');
            padFormula.innerText = `출력 특징 맵 크기 = (5 - 3 + 2×${padding}) / ${stride} + 1 = ${outSize}x${outSize}`;

            const inGrid = document.getElementById('pad-input-grid');
            inGrid.className = `grid gap-1 bg-slate-950 p-1.5 rounded-lg border border-slate-700`;
            inGrid.style.gridTemplateColumns = `repeat(${inSize}, minmax(0, 1fr))`;
            inGrid.innerHTML = '';
            for(let r=0; r<inSize; r++) {
                for(let c=0; c<inSize; c++) {
                    const isZeroPad = isPad && (r===0 || r===inSize-1 || c===0 || c===inSize-1);
                    const cell = document.createElement('div');
                    cell.className = `w-5 h-5 md:w-6 md:h-6 flex items-center justify-center text-[10px] rounded ${isZeroPad ? 'bg-slate-800 text-slate-500 border border-dashed border-slate-600' : 'bg-indigo-900 text-indigo-200'}`;
                    cell.innerText = isZeroPad ? '0' : (inputData[Math.min(4, Math.max(0, r-padding))][Math.min(4, Math.max(0, c-padding))]);
                    inGrid.appendChild(cell);
                }
            }

            const outGrid = document.getElementById('pad-output-grid');
            outGrid.className = `grid gap-1 bg-slate-950 p-1.5 rounded-lg border border-amber-900`;
            outGrid.style.gridTemplateColumns = `repeat(${outSize}, minmax(0, 1fr))`;
            outGrid.innerHTML = '';
            for(let i=0; i<outSize*outSize; i++) {
                const cell = document.createElement('div');
                cell.className = `w-5 h-5 md:w-6 md:h-6 bg-amber-900/50 border border-amber-700 rounded`;
                outGrid.appendChild(cell);
            }
        }

        function renderPool() {
            const inGrid = document.getElementById('pool-input-grid');
            inGrid.innerHTML = '';
            for(let r=0; r<4; r++) {
                for(let c=0; c<4; c++) {
                    const cell = document.createElement('div');
                    cell.id = `pool-in-${r}-${c}`;
                    cell.className = `w-8 h-8 md:w-10 md:h-10 flex items-center justify-center text-sm font-mono rounded bg-slate-800 text-slate-300 border border-slate-700 transition-colors`;
                    cell.innerText = poolInputData[r][c];
                    inGrid.appendChild(cell);
                }
            }
            const outGrid = document.getElementById('pool-output-grid');
            outGrid.innerHTML = '';
            for(let r=0; r<2; r++) {
                for(let c=0; c<2; c++) {
                    const cell = document.createElement('div');
                    cell.id = `pool-out-${r}-${c}`;
                    cell.className = `w-10 h-10 md:w-12 md:h-12 flex items-center justify-center text-base font-bold font-mono rounded bg-slate-900 border border-cyan-900 text-cyan-400 transition-colors`;
                    cell.innerText = poolOutputData[r][c] !== null ? poolOutputData[r][c] : '';
                    outGrid.appendChild(cell);
                }
            }
        }

        function resetPool() {
            poolStep = -1;
            poolOutputData = [[null, null], [null, null]];
            renderPool();
        }

        function nextPoolStep() {
            if(poolStep >= 3) {
                resetPool();
                return;
            }
            poolStep++;
            renderPool();

            const outR = Math.floor(poolStep / 2);
            const outC = poolStep % 2;
            
            let maxVal = -Infinity;
            let maxR = -1;
            let maxC = -1;

            for(let r=0; r<2; r++) {
                for(let c=0; c<2; c++) {
                    const inR = outR * 2 + r;
                    const inC = outC * 2 + c;
                    const val = poolInputData[inR][inC];
                    
                    const cell = document.getElementById(`pool-in-${inR}-${inC}`);
                    cell.classList.replace('bg-slate-800', 'bg-cyan-900');
                    cell.classList.replace('border-slate-700', 'border-cyan-500');
                    
                    if(val > maxVal) {
                        maxVal = val;
                        maxR = inR;
                        maxC = inC;
                    }
                }
            }

            const maxCell = document.getElementById(`pool-in-${maxR}-${maxC}`);
            maxCell.classList.replace('bg-cyan-900', 'bg-cyan-500');
            maxCell.classList.add('text-white', 'font-black', 'scale-110');

            poolOutputData[outR][outC] = maxVal;
            const outCell = document.getElementById(`pool-out-${outR}-${outC}`);
            outCell.innerText = maxVal;
            outCell.classList.replace('bg-slate-900', 'bg-cyan-800');
            outCell.classList.add('scale-105', 'text-white');
        }

        // ================= SECTION 3 LOGIC =================
        function updateLRText(val) {
            learningRate = parseFloat(val);
            document.getElementById('lr-val-text').innerText = learningRate.toFixed(2);
        }

        function renderTrainFilter() {
            renderGrid('train-filter-grid', trainFilter, false);
        }

        // Generate Target Filter from 1st stage User Input Pattern
        function getTargetFilterFromInput() {
            let target = Array(3).fill().map(() => Array(3).fill(0));
            // Crop central 3x3 of user input to serve as ideal feature filter target
            for(let r=0; r<3; r++) {
                for(let c=0; c<3; c++) {
                    target[r][c] = inputData[r+1][c+1] === 1 ? 1.0 : -1.0;
                }
            }
            return target;
        }

        function calculateLoss() {
            const target = getTargetFilterFromInput();
            let totalLoss = 0;
            for(let r=0; r<3; r++) {
                for(let c=0; c<3; c++) {
                    const diff = target[r][c] - trainFilter[r][c];
                    totalLoss += diff * diff;
                }
            }
            return totalLoss / 9;
        }

        function trainStep() {
            epochCount++;
            const target = getTargetFilterFromInput();

            // Gradient Descent towards user-created pattern filter
            for(let r=0; r<3; r++) {
                for(let c=0; c<3; c++) {
                    const grad = -2 * (target[r][c] - trainFilter[r][c]);
                    trainFilter[r][c] -= learningRate * grad;
                    trainFilter[r][c] = Math.max(-2, Math.min(2, trainFilter[r][c]));
                }
            }

            const currentLoss = calculateLoss();
            lossHistory.push(currentLoss);

            renderTrainFilter();
            updateLossChart();
            updatePipelinePrediction(); // Continuous update into Stage 4
        }

        function toggleAutoTrain() {
            const btn = document.getElementById('auto-train-btn');
            if (autoTrainInterval) {
                clearInterval(autoTrainInterval);
                autoTrainInterval = null;
                btn.innerText = "연속 학습 실행";
            } else {
                btn.innerText = "학습 중지";
                let steps = 0;
                autoTrainInterval = setInterval(() => {
                    trainStep();
                    steps++;
                    if (steps >= 20 || calculateLoss() < 0.01) {
                        clearInterval(autoTrainInterval);
                        autoTrainInterval = null;
                        btn.innerText = "연속 학습 실행";
                    }
                }, 150);
            }
        }

        function resetTraining() {
            if (autoTrainInterval) clearInterval(autoTrainInterval);
            autoTrainInterval = null;
            document.getElementById('auto-train-btn').innerText = "연속 학습 실행";
            trainFilter = [
                [0.1, -0.2, 0.0],
                [0.3, 0.1, -0.1],
                [-0.1, 0.2, 0.0]
            ];
            epochCount = 0;
            lossHistory = [];
            if (lossChart) {
                lossChart.data.labels = [];
                lossChart.data.datasets[0].data = [];
                lossChart.update();
            }
            renderTrainFilter();
            updatePipelinePrediction();
        }

        function initLossChart() {
            const ctx = document.getElementById('lossChart').getContext('2d');
            lossChart = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: [],
                    datasets: [{
                        label: '손실값 (Loss)',
                        data: [],
                        borderColor: '#EC4899',
                        backgroundColor: 'rgba(236, 72, 153, 0.1)',
                        borderWidth: 2,
                        fill: true,
                        tension: 0.3
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        x: { display: true, title: { display: true, text: '학습 회수(Epoch)', color: '#94A3B8' }, ticks: { color: '#94A3B8' } },
                        y: { display: true, title: { display: true, text: '오차(Loss)', color: '#94A3B8' }, ticks: { color: '#94A3B8' }, min: 0 }
                    },
                    plugins: {
                        legend: { labels: { color: '#F8FAFC' } }
                    }
                }
            });
        }

        function updateLossChart() {
            if (!lossChart) return;
            lossChart.data.labels.push(epochCount);
            lossChart.data.datasets[0].data.push(lossHistory[lossHistory.length - 1]);
            lossChart.update();
        }

        // ================= SECTION 4 LOGIC =================
        function renderDrawGrid() {
            renderGrid('draw-grid', inputData, true, (r, c) => {
                inputData[r][c] = inputData[r][c] === 1 ? 0 : 1;
                renderDrawGrid();
                syncDataPipeline();
                updatePipelinePrediction();
            });
        }

        function clearDrawGrid() {
            inputData = Array(5).fill().map(() => Array(5).fill(0));
            renderDrawGrid();
            syncDataPipeline();
            updatePipelinePrediction();
        }

        function updatePipelinePrediction() {
            // Calculate similarity score between Stage 4 Grid (inputData) and Learned Stage 3 Filter (trainFilter)
            let matchScore = 0;
            for(let r=0; r<3; r++) {
                for(let c=0; c<3; c++) {
                    const inVal = inputData[r+1][c+1];
                    matchScore += inVal * trainFilter[r][c];
                }
            }

            let loss = calculateLoss();
            let isTrained = loss < 0.2;

            let confidence = Math.min(99, Math.max(5, Math.round((matchScore + 5) * 10)));
            if (!isTrained) confidence = Math.round(confidence * 0.4); // Low confidence if not trained yet

            let otherProb = Math.round((100 - confidence) / 2);

            // Preview active filter text
            const filterPrevEl = document.getElementById('pipeline-filter-preview');
            if (filterPrevEl) {
                filterPrevEl.innerHTML = `현재 가중치 오차: <span class="${isTrained ? 'text-emerald-400 font-bold' : 'text-pink-400'}">${loss.toFixed(3)}</span> (${isTrained ? '학습 완성됨' : '미학습 상태 - 3단계에서 학습 필요'})`;
            }

            const resContainer = document.getElementById('prediction-results');
            resContainer.innerHTML = `
                ${createClassBar('사용자 커스텀 패턴', confidence, isTrained ? 'bg-indigo-500' : 'bg-slate-600')}
                ${createClassBar('기타 다이아몬드/노이즈', otherProb, 'bg-slate-700')}
                ${createClassBar('배경/무작위', 100 - confidence - otherProb, 'bg-slate-800')}
            `;
        }

        function createClassBar(label, prob, colorClass) {
            return `
                <div class="space-y-1 text-xs">
                    <div class="flex justify-between font-semibold">
                        <span class="text-slate-300">${label}</span>
                        <span class="text-slate-100 font-bold">${prob}%</span>
                    </div>
                    <div class="w-full bg-slate-950 h-3 rounded-full overflow-hidden border border-slate-800">
                        <div class="${colorClass} h-full transition-all duration-300" style="width: ${prob}%"></div>
                    </div>
                </div>
            `;
        }

        // INITIALIZATION ON LOAD
        window.onload = () => {
            renderSection1();
            renderTrainFilter();
            renderDrawGrid();
            syncDataPipeline();
        };
    </script>
</body>
</html>
