# --<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>책꽂이 제작 시뮬레이터</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        /* 커스텀 스타일: 책꽂이 부품이 중앙에 배치되도록 설정 */
        .bookshelf-container {
            position: relative;
            width: 300px;
            height: 400px;
            margin: 0 auto;
            border: 1px solid #ccc;
            background-color: #f7f3f0; /* 작업대 배경색 */
            display: flex;
            justify-content: center;
            align-items: flex-end; /* 바닥부터 쌓기 */
            overflow: hidden; /* 망치 애니메이션이 컨테이너를 벗어나지 않도록 */
        }

        /* 기본 목재 스타일 */
        .wood-piece {
            position: absolute;
            background-color: #a0522d; /* 기본 나무색 (SaddleBrown) */
            transition: all 0.5s ease-in-out, background-color 2s ease-in-out;
            border-radius: 4px;
        }

        /* 개별 부품 스타일 */
        #base { width: 100%; height: 20px; bottom: 0; }
        #side-left, #side-right { width: 20px; height: 380px; bottom: 20px; }
        #side-left { left: 0; }
        #side-right { right: 0; }
        .shelf { width: 260px; height: 15px; left: 20px; }
        #shelf-1 { bottom: 120px; }
        #shelf-2 { bottom: 220px; }
        #shelf-3 { bottom: 320px; }

        /* 못 스타일 (크게 수정) */
        .nail {
            position: absolute;
            width: 8px; /* 4px -> 8px (더 크게) */
            height: 8px; /* 4px -> 8px (더 크게) */
            background-color: #6b7280; /* 회색 못 */
            border-radius: 50%;
            opacity: 0;
            transition: opacity 0.3s, transform 0.3s; /* 박히는 효과를 위해 transform 추가 */
            z-index: 10;
        }

        /* 망치 스타일 */
        #hammer {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%) scale(0.1);
            font-size: 50px;
            pointer-events: none; /* 클릭 방지 */
            opacity: 0;
            z-index: 20;
            transition: transform 0.1s ease-out, opacity 0.3s;
        }

        /* 오일 마감 스타일 */
        .oiled {
            background-color: #7b3f00 !important; /* 오일 마감된 나무색 (진한 갈색) */
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.3);
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen p-4 sm:p-8 font-sans">

    <div class="max-w-4xl mx-auto bg-white shadow-xl rounded-xl p-6 sm:p-10">
        <h1 class="text-3xl sm:text-4xl font-extrabold text-center text-gray-800 mb-6 border-b-2 pb-2">
            🪵 책꽂이 제작 시뮬레이터
        </h1>

        <!-- 현재 상태 및 메시지 패널 -->
        <div id="status-panel" class="mb-6 p-4 rounded-lg shadow-inner text-center">
            <p id="current-status" class="text-xl font-semibold text-blue-600">준비: 조립을 시작해 보세요.</p>
            <p id="error-message" class="text-red-500 font-medium mt-2 hidden">순서가 맞지 않습니다!</p>
        </div>

        <div class="flex flex-col lg:flex-row gap-8">
            <!-- 시뮬레이션 영역 -->
            <div class="lg:w-1/2 flex justify-center items-start">
                <div class="bookshelf-container rounded-lg shadow-inner">
                    <!-- 책꽂이 부품들 (초기에는 숨김) -->
                    <div id="base" class="wood-piece shadow-md" style="opacity: 0;"></div>
                    <div id="side-left" class="wood-piece shadow-md" style="opacity: 0;"></div>
                    <div id="side-right" class="wood-piece shadow-md" style="opacity: 0;"></div>
                    <div id="shelf-1" class="wood-piece shelf shadow-md" style="opacity: 0;"></div>
                    <div id="shelf-2" class="wood-piece shelf shadow-md" style="opacity: 0;"></div>
                    <div id="shelf-3" class="wood-piece shelf shadow-md" style="opacity: 0;"></div>

                    <!-- 못 위치 (나중에 추가됨) -->
                    <!-- 못 크기가 커짐에 따라 위치 조정 -->
                    <div id="nail-1" class="nail" style="top: 100px; left: 16px;"></div>
                    <div id="nail-2" class="nail" style="top: 100px; right: 16px;"></div>
                    <div id="nail-3" class="nail" style="top: 300px; left: 16px;"></div>
                    <div id="nail-4" class="nail" style="top: 300px; right: 16px;"></div>
                    <div id="nail-5" class="nail" style="bottom: 15px; left: 100px;"></div>
                    <div id="nail-6" class="nail" style="bottom: 15px; right: 100px;"></div>
                    
                    <!-- 망치 아이콘 추가 -->
                    <div id="hammer" role="img" aria-label="망치 아이콘">🔨</div> 

                </div>
            </div>

            <!-- 컨트롤 및 단계별 버튼 -->
            <div class="lg:w-1/2">
                <div class="space-y-4">
                    <!-- 단계 1: 조립 -->
                    <h2 class="text-2xl font-bold text-gray-700 border-b pb-2">1. 목재 조립 단계</h2>
                    <div id="assembly-buttons" class="grid grid-cols-2 gap-3">
                        <button id="btn-base" class="action-btn bg-yellow-500 hover:bg-yellow-600 text-white font-bold py-3 rounded-lg shadow-md transition duration-200" onclick="performAction('base')">
                            받침대(Base) 조립
                        </button>
                        <button id="btn-side-left" class="action-btn bg-yellow-500 hover:bg-yellow-600 text-white font-bold py-3 rounded-lg shadow-md transition duration-200" onclick="performAction('side-left')" disabled>
                            왼쪽 측면 조립
                        </button>
                        <button id="btn-shelf-1" class="action-btn bg-yellow-500 hover:bg-yellow-600 text-white font-bold py-3 rounded-lg shadow-md transition duration-200" onclick="performAction('shelf-1')" disabled>
                            선반 1 조립
                        </button>
                        <button id="btn-shelf-2" class="action-btn bg-yellow-500 hover:bg-yellow-600 text-white font-bold py-3 rounded-lg shadow-md transition duration-200" onclick="performAction('shelf-2')" disabled>
                            선반 2 조립
                        </button>
                        <button id="btn-shelf-3" class="action-btn bg-yellow-500 hover:bg-yellow-600 text-white font-bold py-3 rounded-lg shadow-md transition duration-200" onclick="performAction('shelf-3')" disabled>
                            선반 3 조립
                        </button>
                        <button id="btn-side-right" class="action-btn bg-yellow-500 hover:bg-yellow-600 text-white font-bold py-3 rounded-lg shadow-md transition duration-200" onclick="performAction('side-right')" disabled>
                            오른쪽 측면 조립
                        </button>
                    </div>

                    <!-- 단계 2: 못질 -->
                    <h2 class="text-2xl font-bold text-gray-700 border-b pb-2 mt-6">2. 못질 단계</h2>
                    <button id="btn-nailing" class="action-btn w-full bg-red-600 hover:bg-red-700 text-white font-bold py-4 rounded-lg shadow-md transition duration-200" onclick="performAction('nailing')" disabled>
                        🔨 망치로 못 박기
                    </button>
                    <p id="nail-status" class="text-sm text-gray-500 text-center mt-1">남은 못: 6개</p>


                    <!-- 단계 3: 마감 -->
                    <h2 class="text-2xl font-bold text-gray-700 border-b pb-2 mt-6">3. 마감 단계</h2>
                    <button id="btn-oiling" class="action-btn w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-4 rounded-lg shadow-md transition duration-200" onclick="performAction('oiling')" disabled>
                        🧴 오일 바르기 (마감)
                    </button>
                </div>
            </div>
        </div>

        <!-- 초기화 버튼 -->
        <div class="mt-8 text-center">
            <button id="btn-reset" class="bg-gray-500 hover:bg-gray-600 text-white font-bold py-2 px-6 rounded-lg shadow-lg transition duration-200" onclick="resetSimulator()">
                시뮬레이터 초기화ㅊ
            </button>
        </div>
    </div>

    <script>
        // 시뮬레이터 상태 관리 객체
        let simulationState = {
            currentStep: 'assembly', // 'assembly', 'nailing', 'oiling', 'finished'
            assemblySequence: [
                'base', 'side-left', 'shelf-1', 'shelf-2', 'shelf-3', 'side-right'
            ],
            assemblyIndex: 0,
            nailsDriven: 0,
            totalNails: 6
        };

        // DOM 요소 캐싱
        const elements = {
            status: document.getElementById('current-status'),
            error: document.getElementById('error-message'),
            nailStatus: document.getElementById('nail-status'),
            nailingBtn: document.getElementById('btn-nailing'),
            oilingBtn: document.getElementById('btn-oiling'),
            hammer: document.getElementById('hammer'), // 해머 요소 추가
            parts: {}
        };

        // 모든 목재 부품 ID
        const partIds = ['base', 'side-left', 'side-right', 'shelf-1', 'shelf-2', 'shelf-3'];
        partIds.forEach(id => { elements.parts[id] = document.getElementById(id); });

        // 모든 못 ID
        const nailIds = ['nail-1', 'nail-2', 'nail-3', 'nail-4', 'nail-5', 'nail-6'];
        nailIds.forEach(id => { elements.parts[id] = document.getElementById(id); });


        // 부품 조립 버튼 상태를 업데이트하는 함수
        function updateAssemblyButtons() {
            // 모든 조립 버튼 비활성화 및 기본 스타일로 설정
            document.querySelectorAll('#assembly-buttons button').forEach(btn => {
                btn.disabled = true;
                btn.classList.remove('bg-green-500', 'hover:bg-green-600');
                btn.classList.add('bg-gray-300', 'cursor-not-allowed');
            });

            // 현재 순서의 버튼만 활성화
            if (simulationState.currentStep === 'assembly') {
                const nextPartId = simulationState.assemblySequence[simulationState.assemblyIndex];
                if (nextPartId) {
                    const nextBtn = document.getElementById(`btn-${nextPartId}`);
                    if (nextBtn) {
                        nextBtn.disabled = false;
                        nextBtn.classList.remove('bg-gray-300', 'cursor-not-allowed');
                        nextBtn.classList.add('bg-yellow-500', 'hover:bg-yellow-600');
                    }
                }
            } else if (simulationState.currentStep === 'nailing') {
                // 조립 완료 후에는 조립 버튼 영구 비활성화
                document.querySelectorAll('#assembly-buttons button').forEach(btn => {
                    btn.classList.remove('bg-yellow-500', 'hover:bg-yellow-600');
                    btn.classList.add('bg-green-500', 'cursor-default'); // 조립 완료 표시
                    btn.disabled = true;
                });
            }
        }

        // 전체 UI 상태를 업데이트하는 함수
        function updateUI() {
            elements.error.classList.add('hidden'); // 에러 메시지 숨김

            // 1. 조립 단계 버튼 업데이트
            updateAssemblyButtons();

            // 2. 못질 단계 버튼 업데이트
            elements.nailingBtn.disabled = simulationState.currentStep !== 'nailing';
            elements.nailingBtn.classList.toggle('bg-gray-300', simulationState.currentStep !== 'nailing');
            elements.nailingBtn.classList.toggle('bg-red-600', simulationState.currentStep === 'nailing');

            // 3. 오일링 단계 버튼 업데이트
            elements.oilingBtn.disabled = simulationState.currentStep !== 'oiling';
            elements.oilingBtn.classList.toggle('bg-gray-300', simulationState.currentStep !== 'oiling');
            elements.oilingBtn.classList.toggle('bg-blue-600', simulationState.currentStep === 'oiling');

            // 4. 상태 메시지 업데이트
            let statusText = '';
            switch(simulationState.currentStep) {
                case 'assembly':
                    statusText = `조립 중: 다음 부품인 ${getPartName(simulationState.assemblySequence[simulationState.assemblyIndex])}을(를) 조립하세요.`;
                    break;
                case 'nailing':
                    const nailsLeft = simulationState.totalNails - simulationState.nailsDriven;
                    statusText = `못질 중: 망치로 못을 박아 책꽂이를 단단하게 고정하세요. (남은 못: ${nailsLeft}개)`;
                    elements.nailStatus.textContent = `남은 못: ${nailsLeft}개`;
                    break;
                case 'oiling':
                    statusText = '마감 중: 이제 오일을 발라 책꽂이를 보호하고 색을 입힐 차례입니다.';
                    elements.nailStatus.textContent = `못질 완료!`;
                    break;
                case 'finished':
                    statusText = '🎉 제작 완료! 멋진 책꽂이가 완성되었습니다.';
                    elements.nailStatus.textContent = `최종 마감 완료!`;
                    break;
            }
            elements.status.textContent = statusText;
        }

        // 부품 ID를 한국어 이름으로 변환하는 헬퍼 함수
        function getPartName(id) {
            switch(id) {
                case 'base': return '받침대';
                case 'side-left': return '왼쪽 측면판';
                case 'side-right': return '오른쪽 측면판';
                case 'shelf-1': return '첫 번째 선반';
                case 'shelf-2': return '두 번째 선반';
                case 'shelf-3': return '세 번째 선반';
                default: return '알 수 없는 부품';
            }
        }
        
        // 망치질 애니메이션을 시뮬레이션하는 함수
        function simulateHammering(nailElement) {
            // 못의 컨테이너 내 상대 위치 계산 (중앙을 기준으로)
            const nailX = nailElement.offsetLeft + (nailElement.offsetWidth / 2);
            const nailY = nailElement.offsetTop + (nailElement.offsetHeight / 2);

            // 1. 망치를 못 위치로 이동시키고 크게 보이게 함
            elements.hammer.style.opacity = 1;
            // X, Y 위치로 이동 (중앙 정렬 보정)
            elements.hammer.style.transform = `translate(calc(${nailX}px - 50%), calc(${nailY}px - 50%)) scale(1.5)`; 

            // 2. 망치가 내리치는 애니메이션
            setTimeout(() => {
                // 망치 크기를 작게 하여 타격 시뮬레이션
                elements.hammer.style.transform = `translate(calc(${nailX}px - 50%), calc(${nailY}px - 50%)) scale(1)`; 
                
                // 못이 박히는 효과 (아래로 이동하고 투명도 1)
                nailElement.style.transform = 'translateY(1px)'; 
                nailElement.style.opacity = 1; 
            }, 100);

            // 3. 망치를 숨김
            setTimeout(() => {
                elements.hammer.style.opacity = 0;
                // 다음 못질을 위해 크기를 원래대로 돌려놓음
                elements.hammer.style.transform = `translate(-50%, -50%) scale(0.1)`; 
            }, 400);
        }

        // 모든 시뮬레이션 액션을 처리하는 메인 함수
        function performAction(actionType) {
            elements.error.classList.add('hidden');

            // --- 조립 단계 처리 ---
            if (simulationState.currentStep === 'assembly') {
                const expectedPart = simulationState.assemblySequence[simulationState.assemblyIndex];

                if (actionType === expectedPart) {
                    // 올바른 순서: 부품 표시
                    elements.parts[actionType].style.opacity = 1;
                    
                    // 다음 순서로 이동
                    simulationState.assemblyIndex++;

                    if (simulationState.assemblyIndex >= simulationState.assemblySequence.length) {
                        // 조립 완료 -> 못질 단계로 전환
                        simulationState.currentStep = 'nailing';
                        elements.status.textContent = '✅ 조립 완료! 이제 못질을 시작하세요.';
                    }
                } else {
                    // 잘못된 순서: 에러 표시
                    elements.error.textContent = `순서가 맞지 않습니다! 다음은 ${getPartName(expectedPart)} 조립 차례입니다.`;
                    elements.error.classList.remove('hidden');
                }
            }
            // --- 못질 단계 처리 ---
            else if (simulationState.currentStep === 'nailing' && actionType === 'nailing') {
                if (simulationState.nailsDriven < simulationState.totalNails) {
                    
                    const nailId = nailIds[simulationState.nailsDriven];
                    const nailElement = elements.parts[nailId];
                    
                    // 망치질 애니메이션 실행
                    simulateHammering(nailElement);
                    
                    // 상태 업데이트는 애니메이션 후에 진행
                    // 버튼 연타를 막기 위해 잠시 버튼을 비활성화
                    elements.nailingBtn.disabled = true;

                    setTimeout(() => {
                        simulationState.nailsDriven++;
                        
                        if (simulationState.nailsDriven >= simulationState.totalNails) {
                            // 못질 완료 -> 오일링 단계로 전환
                            simulationState.currentStep = 'oiling';
                            elements.status.textContent = '✅ 못질 완료! 이제 오일 마감을 하세요.';
                        } else {
                            // 못질이 끝나지 않았으면 버튼 다시 활성화
                            elements.nailingBtn.disabled = false;
                        }
                        updateUI(); // UI 상태 업데이트
                    }, 500); // 애니메이션 시간 고려하여 0.5초 후 상태 업데이트
                    return; // setTimeout 내에서 상태 업데이트가 이루어지므로 여기서 리턴
                }
            }
            // --- 오일링 단계 처리 ---
            else if (simulationState.currentStep === 'oiling' && actionType === 'oiling') {
                // 오일 마감 시뮬레이션: 모든 부품에 클래스 추가
                partIds.forEach(id => {
                    elements.parts[id].classList.add('oiled');
                });
                // 못도 오일로 인해 색이 변했다고 가정
                nailIds.forEach(id => {
                    elements.parts[id].style.backgroundColor = '#9a7b4f'; // 어두운 금속색
                });

                // 최종 완료 단계로 전환
                simulationState.currentStep = 'finished';
                elements.status.textContent = '🎉 제작 완료! 멋진 오일 마감 책꽂이가 완성되었습니다.';
            }

            updateUI();
        }

        // 시뮬레이터 초기화 함수
        function resetSimulator() {
            // 상태 초기화
            simulationState = {
                currentStep: 'assembly',
                assemblySequence: ['base', 'side-left', 'shelf-1', 'shelf-2', 'shelf-3', 'side-right'],
                assemblyIndex: 0,
                nailsDriven: 0,
                totalNails: 6
            };

            // UI 요소 초기화
            partIds.forEach(id => {
                const part = elements.parts[id];
                part.style.opacity = 0;
                part.classList.remove('oiled');
            });

            nailIds.forEach(id => {
                const nail = elements.parts[id];
                nail.style.opacity = 0;
                nail.style.backgroundColor = '#6b7280';
                nail.style.transform = 'translateY(0)'; // 박힘 효과 초기화
            });

            // 망치 아이콘 초기화
            elements.hammer.style.opacity = 0;
            elements.hammer.style.transform = `translate(-50%, -50%) scale(0.1)`; 


            // 초기 상태로 UI 업데이트
            updateUI();
        }

        // 초기 로드 시 시뮬레이터 설정
        window.onload = function() {
            resetSimulator();
            // 모든 조립 버튼에 초기 스타일 적용을 위해 한 번 더 호출
            updateAssemblyButtons(); 
        };
    </script>

</body>
</html>
