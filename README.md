<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>nibir · romantic call</title>
    <script src="/socket.io/socket.io.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            background: #0a0c0f;
            font-family: system-ui, -apple-system, sans-serif;
            display: flex;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            color: #e2e8f0;
            padding: 16px;
        }

        .app {
            max-width: 420px;
            width: 100%;
            background: #141a20;
            border-radius: 40px;
            padding: 24px 20px;
            box-shadow: 0 20px 40px #00000080;
            border: 1px solid #2c3743;
        }

        /* Welcome */
        .welcome {
            display: flex;
            flex-direction: column;
            gap: 32px;
        }

        .welcome h2 {
            font-size: 28px;
            font-weight: 300;
            text-align: center;
            color: #d6e3f5;
        }

        .welcome h2 i {
            color: #c87958;
            margin-right: 10px;
        }

        .input-group {
            display: flex;
            flex-direction: column;
            gap: 16px;
        }

        .input-field {
            background: #1f2933;
            border: 1px solid #3d4e5f;
            border-radius: 60px;
            padding: 16px 24px;
            font-size: 18px;
            color: white;
            outline: none;
            text-align: center;
        }

        .input-field:focus {
            border-color: #7f9bc0;
        }

        .btn {
            background: #26333f;
            border: 1px solid #405a72;
            border-radius: 60px;
            padding: 16px;
            font-size: 18px;
            color: white;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
            transition: 0.2s;
        }

        .btn:hover {
            background: #2f4051;
        }

        /* Users */
        .users-screen {
            display: none;
            flex-direction: column;
            gap: 20px;
        }

        .users-screen.show {
            display: flex;
        }

        .user-header {
            display: flex;
            justify-content: space-between;
            color: #a2b9d4;
            font-size: 14px;
            letter-spacing: 1px;
        }

        .users-list {
            display: flex;
            flex-direction: column;
            gap: 10px;
            max-height: 400px;
            overflow-y: auto;
        }

        .user-item {
            background: #1d2832;
            border: 1px solid #33495c;
            border-radius: 50px;
            padding: 16px 20px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            cursor: pointer;
            transition: 0.2s;
        }

        .user-item:hover {
            background: #263644;
            border-color: #52739b;
            transform: translateX(5px);
        }

        .user-item .name {
            display: flex;
            align-items: center;
            gap: 12px;
            font-size: 18px;
        }

        .online-dot {
            width: 10px;
            height: 10px;
            background: #4ade80;
            border-radius: 50%;
            display: inline-block;
        }

        .call-badge {
            background: #2e4051;
            padding: 6px 18px;
            border-radius: 40px;
            font-size: 14px;
        }

        .empty-state {
            text-align: center;
            padding: 40px;
            color: #6c8bb0;
            font-style: italic;
        }

        /* Call */
        .call-screen {
            display: none;
            flex-direction: column;
            align-items: center;
            gap: 30px;
        }

        .call-screen.show {
            display: flex;
        }

        .avatar {
            width: 120px;
            height: 120px;
            background: #23303d;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            border: 2px solid #6889b0;
        }

        .avatar i {
            font-size: 60px;
            color: #bed6f5;
        }

        .caller-name {
            font-size: 36px;
            font-weight: 300;
            color: #e7f0fd;
        }

        .call-status {
            color: #95b2d3;
            font-size: 14px;
            margin-top: 5px;
        }

        .controls {
            display: flex;
            flex-wrap: wrap;
            gap: 15px;
            justify-content: center;
            margin-top: 20px;
        }

        .control-btn {
            background: #263746;
            border: 1.5px solid #4b6c8a;
            border-radius: 50px;
            padding: 14px 28px;
            font-size: 18px;
            color: white;
            display: flex;
            align-items: center;
            gap: 10px;
            cursor: pointer;
            min-width: 140px;
            justify-content: center;
        }

        .control-btn.active {
            background: #2d4259;
            border-color: #8aafe0;
        }

        .control-btn.reject {
            background: #2e1f26;
            border-color: #b1465c;
            color: #ffd3de;
        }

        .footer {
            margin-top: 20px;
            color: #56738f;
            font-size: 13px;
            border-top: 1px dashed #38506b;
            padding-top: 20px;
            text-align: center;
        }

        /* Incoming */
        .incoming-popup {
            position: fixed;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: #1f2e3d;
            border: 2px solid #7196c2;
            border-radius: 60px;
            padding: 20px 30px;
            box-shadow: 0 10px 30px black;
            z-index: 1000;
            display: none;
            animation: slideDown 0.3s;
        }

        .incoming-popup.show {
            display: block;
        }

        @keyframes slideDown {
            from { top: -100px; }
            to { top: 20px; }
        }
    </style>
</head>
<body>
    <div class="app">
        <!-- Welcome -->
        <div id="welcomeScreen" class="welcome">
            <h2><i class="fa-regular fa-heart"></i> enter with your name</h2>
            <div class="input-group">
                <input type="text" id="nameInput" class="input-field" placeholder="your name...">
                <button class="btn" id="joinBtn"><i class="fa-regular fa-paper-plane"></i> enter</button>
            </div>
        </div>

        <!-- Users -->
        <div id="usersScreen" class="users-screen">
            <div class="user-header">
                <span><i class="fa-regular fa-star"></i> online</span>
                <span id="onlineCount">0</span>
            </div>
            <div class="users-list" id="usersList"></div>
            <div class="footer">click name to call</div>
        </div>

        <!-- Call -->
        <div id="callScreen" class="call-screen">
            <div class="avatar">
                <i class="fa-regular fa-user"></i>
            </div>
            <div>
                <div class="caller-name" id="callerName">---</div>
                <div class="call-status" id="callStatus">waiting...</div>
            </div>
            <div class="controls">
                <button class="control-btn" id="speakerBtn"><i class="fa-solid fa-volume-high"></i> speaker</button>
                <button class="control-btn active" id="micBtn"><i class="fa-solid fa-microphone"></i> mic</button>
                <button class="control-btn reject" id="endCallBtn"><i class="fa-solid fa-phone-slash"></i> end</button>
            </div>
            <div class="footer">false caller by <strong>nibir</strong></div>
        </div>
    </div>

    <!-- Incoming -->
    <div id="incomingPopup" class="incoming-popup">
        <div style="display: flex; gap: 20px; align-items: center;">
            <span id="callerNamePopup"></span> is calling ❤️
            <button onclick="acceptIncoming()" style="background: #1e4620; border: none; color: white; padding: 8px 20px; border-radius: 40px; cursor: pointer;">accept</button>
            <button onclick="rejectIncoming()" style="background: #462020; border: none; color: white; padding: 8px 20px; border-radius: 40px; cursor: pointer;">reject</button>
        </div>
    </div>

    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0-beta3/css/all.min.css">

    <script>
        const socket = io();
        
        // State
        let myName = localStorage.getItem('myName') || '';
        let currentCall = null;
        let peerConnection = null;
        let localStream = null;
        
        // Audio elements
        const remoteAudio = document.createElement("audio");
remoteAudio.autoplay = true;
remoteAudio.playsInline = true;
remoteAudio.controls = false;
remoteAudio.style.display = "none";
document.body.appendChild(remoteAudio);

        // DOM
        const welcomeScreen = document.getElementById('welcomeScreen');
        const usersScreen = document.getElementById('usersScreen');
        const callScreen = document.getElementById('callScreen');
        const nameInput = document.getElementById('nameInput');
        const joinBtn = document.getElementById('joinBtn');
        const usersList = document.getElementById('usersList');
        const onlineCount = document.getElementById('onlineCount');
        const callerNameSpan = document.getElementById('callerName');
        const callStatusSpan = document.getElementById('callStatus');
        const speakerBtn = document.getElementById('speakerBtn');
        const micBtn = document.getElementById('micBtn');
        const endCallBtn = document.getElementById('endCallBtn');
        const incomingPopup = document.getElementById('incomingPopup');
        const callerNamePopup = document.getElementById('callerNamePopup');

        // Auto join if name exists
        if (myName) {
            welcomeScreen.style.display = 'none';
            usersScreen.classList.add('show');
            socket.emit('join', myName);
        }

        // Join button
        joinBtn.addEventListener('click', () => {
            const name = nameInput.value.trim();
            if (!name) return alert('Please enter your name');
            myName = name;
            localStorage.setItem('myName', name);
            welcomeScreen.style.display = 'none';
            usersScreen.classList.add('show');
            socket.emit('join', name);
        });

        nameInput.addEventListener('keypress', (e) => {
            if (e.key === 'Enter') joinBtn.click();
        });

        // Socket events
        socket.on('users-list', (users) => {
            renderUsers(users);
        });

        socket.on('user-joined', (user) => {
            addUser(user.name);
        });

        socket.on('user-left', (user) => {
            removeUser(user.name);
        });

        socket.on('incoming-call', (data) => {
            callerNamePopup.textContent = data.from;
            incomingPopup.classList.add('show');
            currentCall = {
                type: 'incoming',
                callerSocketId: data.callerSocketId,
                callerName: data.from
            };
        });

        socket.on('call-accepted', async (data) => {
            incomingPopup.classList.remove('show');
            callStatusSpan.textContent = 'connected';
            currentCall = {
                ...currentCall,
                accepted: true,
                targetSocketId: data.accepterSocketId
            };
            
            // Create offer
            await createPeerConnection(data.accepterSocketId);
            
            try {
                const offer = await peerConnection.createOffer();
                await peerConnection.setLocalDescription(offer);
                socket.emit('offer', {
                    target: data.accepterSocketId,
                    offer: offer
                });
            } catch (err) {
                console.error('Offer error:', err);
            }
        });

        socket.on('call-rejected', () => {
            alert('Call was rejected');
            endCall();
        });

        socket.on('call-error', (msg) => {
            alert(msg);
            endCall();
        });

        socket.on('offer', async (data) => {
            await createPeerConnection(data.from);
            
            try {
                await peerConnection.setRemoteDescription(new RTCSessionDescription(data.offer));
                const answer = await peerConnection.createAnswer();
                await peerConnection.setLocalDescription(answer);
                socket.emit('answer', {
                    target: data.from,
                    answer: answer
                });
            } catch (err) {
                console.error('Answer error:', err);
            }
        });

        socket.on('answer', async (data) => {
            try {
                await peerConnection.setRemoteDescription(new RTCSessionDescription(data.answer));
            } catch (err) {
                console.error('Set answer error:', err);
            }
        });

        socket.on('ice-candidate', async (data) => {
            if (peerConnection) {
                try {
                    await peerConnection.addIceCandidate(new RTCIceCandidate(data.candidate));
                } catch (err) {
                    console.error('ICE error:', err);
                }
            }
        });

        // Render users
        function renderUsers(users) {
            if (!users || users.length === 0) {
                usersList.innerHTML = '<div class="empty-state">no one else online</div>';
                onlineCount.textContent = '0';
                return;
            }
            onlineCount.textContent = users.length;
            let html = '';
            users.forEach(user => {
                html += `<div class="user-item" onclick="startCall('${user.name}')">
                    <span class="name"><span class="online-dot"></span> <i class="fa-regular fa-circle-user"></i> ${user.name}</span>
                    <span class="call-badge"><i class="fa-regular fa-phone"></i> call</span>
                </div>`;
            });
            usersList.innerHTML = html;
        }

        function addUser(name) {
            if (document.querySelector(`[onclick="startCall('${name}')"]`)) return;
            const div = document.createElement('div');
            div.className = 'user-item';
            div.setAttribute('onclick', `startCall('${name}')`);
            div.innerHTML = `<span class="name"><span class="online-dot"></span> <i class="fa-regular fa-circle-user"></i> ${name}</span>
                <span class="call-badge"><i class="fa-regular fa-phone"></i> call</span>`;
            usersList.appendChild(div);
            onlineCount.textContent = parseInt(onlineCount.textContent) + 1;
        }

        function removeUser(name) {
            const el = document.querySelector(`[onclick="startCall('${name}')"]`);
            if (el) {
                el.remove();
                onlineCount.textContent = parseInt(onlineCount.textContent) - 1;
            }
        }

        // Start call
        window.startCall = async (targetName) => {
            if (targetName === myName) return alert("Can't call yourself");
            
            try {
                // Get microphone
                localStream = await navigator.mediaDevices.getUserMedia({ 
  audio: {
    echoCancellation: true,
    noiseSuppression: true,
    autoGainControl: true,
    channelCount: 1,
    sampleRate: 48000,
    latency: 0
  } 
});
                
                callerNameSpan.textContent = targetName;
                callStatusSpan.textContent = 'calling...';
                usersScreen.classList.remove('show');
                callScreen.classList.add('show');
                
                currentCall = {
                    type: 'outgoing',
                    targetName: targetName,
                    accepted: false
                };
                
                socket.emit('call-user', {
                    targetName: targetName,
                    callerName: myName
                });
                
            } catch (err) {
                alert('Microphone access needed for calls');
                console.error('Media error:', err);
            }
        };

        // Accept incoming
        window.acceptIncoming = async () => {
            incomingPopup.classList.remove('show');
            
            try {
                localStream = await navigator.mediaDevices.getUserMedia({ 
  audio: {
    echoCancellation: true,
    noiseSuppression: true,
    autoGainControl: true,
    channelCount: 1,
    sampleRate: 48000,
    latency: 0
  } 
});
                
                callerNameSpan.textContent = currentCall.callerName;
                callStatusSpan.textContent = 'connected';
                usersScreen.classList.remove('show');
                callScreen.classList.add('show');
                
                socket.emit('accept-call', {
                    callerSocketId: currentCall.callerSocketId,
                    accepterName: myName
                });
                
                currentCall.accepted = true;
                
            } catch (err) {
                alert('Microphone access needed');
                console.error('Media error:', err);
            }
        };

        // Reject incoming
        window.rejectIncoming = () => {
            socket.emit('reject-call', {
                callerSocketId: currentCall.callerSocketId
            });
            incomingPopup.classList.remove('show');
            currentCall = null;
        };

        // Create peer connection
async function createPeerConnection(targetSocketId) {

    peerConnection = new RTCPeerConnection({
        iceServers: [
            { urls: "stun:stun.l.google.com:19302" },
            { urls: "stun:stun1.l.google.com:19302" },
            { urls: "stun:stun2.l.google.com:19302" }
        ],
        sdpSemantics: "unified-plan"
    });
            
            // Add local stream
            if (localStream) {
                localStream.getTracks().forEach(track => {
                    peerConnection.addTrack(track, localStream);
                });
            }
            
            // Handle remote stream
            peerConnection.ontrack = (event) => {
                console.log('Got remote track');
                if (remoteAudio.srcObject !== event.streams[0]) {
                    remoteAudio.srcObject = event.streams[0];
remoteAudio.play().catch(()=>{});
                    // Play with user interaction
                    document.addEventListener('click', function playAudio() {
                        remoteAudio.play();
                        document.removeEventListener('click', playAudio);
                    }, { once: true });
                }
            };
            
            // ICE candidates
            peerConnection.onicecandidate = (event) => {
                if (event.candidate) {
                    socket.emit('ice-candidate', {
                        target: targetSocketId,
                        candidate: event.candidate
                    });
                }
            };
            
            // Connection state
            peerConnection.onconnectionstatechange = () => {
                console.log('Connection state:', peerConnection.connectionState);
                if (peerConnection.connectionState === 'disconnected' || 
                    peerConnection.connectionState === 'failed') {
                    endCall();
                }
            };
            
            return peerConnection;
        }

        // Speaker button
speakerBtn.addEventListener('click', () => {

    speakerBtn.classList.toggle('active');

    if (speakerBtn.classList.contains('active')) {
        // Speaker ON
        remoteAudio.volume = 1;
    } else {
        // Speaker OFF (earpiece-like low volume)
        remoteAudio.volume = 0.25;
    }

});

        // Mic button
        micBtn.addEventListener('click', () => {
            micBtn.classList.toggle('active');
            if (localStream) {
                localStream.getAudioTracks().forEach(track => {
                    track.enabled = micBtn.classList.contains('active');
                });
            }
        });

        // End call
        endCallBtn.addEventListener('click', endCall);

        function endCall() {
            if (peerConnection) {
                peerConnection.close();
                peerConnection = null;
            }
            if (localStream) {
                localStream.getTracks().forEach(t => {
                    t.stop();
                    localStream.removeTrack(t);
                });
                localStream = null;
            }
            remoteAudio.srcObject = null;
            currentCall = null;
            callScreen.classList.remove('show');
            usersScreen.classList.add('show');
            micBtn.classList.add('active');
            speakerBtn.classList.remove('active');
        }
    </script>
</body>
</html>
