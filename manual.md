OSS 메뉴얼작성 프로젝트

CLOVA 메뉴얼
==================

###### 조장 : 김민재, 조원 : 정준영, 서대원

******************
목차
==================
1. CLOVA 란?
2. CSR
3. CSS
4. CFR
5. UX 고려사항
******************
CLOVA 란?
==================
콘텐츠 개발자 혹은 회사들이 음성기반 기능을 구현할 수 있도록 도와준다. 개발자들이 간단한 설정 및 코드 작성 만으로도 짧은 시간안에 당신만의 App에 Clova에 추가할 수 있다. 자연어 처리에 대해서 잘 알지 못하더라도 새로운 음성 기반의 서비스를 구현 가능하게 해준다. Clova Extensions Kit은 무료로 제공되며 더 많은 사용자들과 만날 수 있는 기회를 제공할 것이다.

******************
CSR
==================

CSR API란?
------------------
CSR은 Clova Speech Recognition의 약자로써 음성 입력을 스트리밍 형태로 입력받은 후 음성 인식한 결과를 텍스트로 반환한다. CSR API는 사용자 음성 입력을 전달 받기 위해 자체 개발한 스트리밍 프로토콜을 사용중이다 따라서, HTTP기반의 REST API형태가 아니라 Android SDK형태로 CSR API를 제공하고 있다.

사전 준비사항
------------------
CSR API를 사용하려면 개발하려는 애플리케이션을 네이버 개발자 센터에 등록해야 한다. 이때, 사용할 API에 대한 권한을 설정해야하며 ,API 사용 시 필요한 인증 정보를 획득해야 합니다. 애플리케이션 등록을 참고하여 다음과 같이 필요한 사항을 미리 준비해야한다.

    1. 애플리케이션 등록하기를 통해 앱 등록 페이지로 이동합니다.
    2. 사용 API에서 음성인식(CSR API)을 선택합니다.
    3. 비로그인 오픈 API 서비스 환경에 개발하는 앱 정보를 입력합니다.
    4. 등록하기 버튼을 클릭합니다.
    
API 사용하기
------------------
1. 다음 구문을 app/build.gradle 파일에 추가한다.
![Alt text](./img/1.png)
2. 다음과 같이 Android Manifest파일을 설정한다.
<pre>
    * 패키지 이름 : manifest 요소의 package 속성 값이 사전 준비사항에서 등록한 안드로이드 앱 패키지 이름과 같아야 합니다.
    
    * 권한 설정 : 사용자의 음성 입력을 마이크를 통해 녹음해야 하고 녹음된 데이터를 서버로 전송해야 합니다. 따라서, android.permission.INTERNET와 android.permission.RECORD_AUDIO에 대한 권한이 반드시 필요합니다.
</pre>
![Alt text](./img/2.png)
3. (선택) proguard-rules.pro 파일에 다음을 추가합니다. 아래 코드는 앱을 보다 가볍고 안전하게 만들어줍니다.
![Alt text](./img/3.png)

구현 예제
-------------------
<code>
   
    public class MainActivity extends Activity {
	private static final String TAG = MainActivity.class.getSimpleName();
	private static final String CLIENT_ID = "YOUR CLIENT ID"; // "내 애플리케이션"에서 Client ID를 확인해서 이곳에 적어주세요.
    private RecognitionHandler handler;
    private NaverRecognizer naverRecognizer;
    private TextView txtResult;
    private Button btnStart;
    private String mResult;
    private AudioWriterPCM writer;
    // Handle speech recognition Messages.
    private void handleMessage(Message msg) {
        switch (msg.what) {
            case R.id.clientReady: // 음성인식 준비 가능
                txtResult.setText("Connected");
                writer = new AudioWriterPCM(Environment.getExternalStorageDirectory().getAbsolutePath() + "/NaverSpeechTest");
                writer.open("Test");
                break;
            case R.id.audioRecording:
                writer.write((short[]) msg.obj);
                break;
            case R.id.partialResult:
                mResult = (String) (msg.obj);
                txtResult.setText(mResult);
                break;
            case R.id.finalResult: // 최종 인식 결과
            	SpeechRecognitionResult speechRecognitionResult = (SpeechRecognitionResult) msg.obj;
            	List<String> results = speechRecognitionResult.getResults();
            	StringBuilder strBuf = new StringBuilder();
            	for(String result : results) {
            		strBuf.append(result);
            		strBuf.append("\n");
            	}
                mResult = strBuf.toString();
                txtResult.setText(mResult);
                break;
            case R.id.recognitionError:
                if (writer != null) {
                    writer.close();
                }
                mResult = "Error code : " + msg.obj.toString();
                txtResult.setText(mResult);
                btnStart.setText(R.string.str_start);
                btnStart.setEnabled(true);
                break;
            case R.id.clientInactive:
                if (writer != null) {
                    writer.close();
                }
                btnStart.setText(R.string.str_start);
                btnStart.setEnabled(true);
                break;
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        txtResult = (TextView) findViewById(R.id.txt_result);
        btnStart = (Button) findViewById(R.id.btn_start);
        handler = new RecognitionHandler(this);
        naverRecognizer = new NaverRecognizer(this, handler, CLIENT_ID);
        btnStart.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if(!naverRecognizer.getSpeechRecognizer().isRunning()) {
                    mResult = "";
                    txtResult.setText("Connecting...");
                    btnStart.setText(R.string.str_stop);
                    naverRecognizer.recognize();
                } else {
                    Log.d(TAG, "stop and wait Final Result");
                    btnStart.setEnabled(false);
                    naverRecognizer.getSpeechRecognizer().stop();
                }
            }
        });
    }
    @Override
    protected void onStart() {
    	super.onStart(); // 음성인식 서버 초기화는 여기서
    	naverRecognizer.getSpeechRecognizer().initialize();
    }
    @Override
    protected void onResume() {
        super.onResume();
        mResult = "";
        txtResult.setText("");
        btnStart.setText(R.string.str_start);
        btnStart.setEnabled(true);
    }
    @Override
    protected void onStop() {
    	super.onStop(); // 음성인식 서버 종료
    	naverRecognizer.getSpeechRecognizer().release();
    }
    // Declare handler for handling SpeechRecognizer thread's Messages.
    static class RecognitionHandler extends Handler {
        private final WeakReference<MainActivity> mActivity;
        RecognitionHandler(MainActivity activity) {
            mActivity = new WeakReference<MainActivity>(activity);
        }
        @Override
        public void handleMessage(Message msg) {
            MainActivity activity = mActivity.get();
            if (activity != null) {
                activity.handleMessage(msg);
            }
        }
    }
    }
    class NaverRecognizer implements SpeechRecognitionListener {
        private final static String TAG = NaverRecognizer.class.getSimpleName();
        private Handler mHandler;
        private SpeechRecognizer mRecognizer;
        public NaverRecognizer(Context context, Handler handler, String clientId) {
            this.mHandler = handler;
            try {
                mRecognizer = new SpeechRecognizer(context, clientId);
            } catch (SpeechRecognitionException e) {
                e.printStackTrace();
            }
            mRecognizer.setSpeechRecognitionListener(this);
        }
        public SpeechRecognizer getSpeechRecognizer() {
            return mRecognizer;
        }
        public void recognize() {
            try {
                mRecognizer.recognize(new SpeechConfig(LanguageType.KOREAN, EndPointDetectType.AUTO));
            } catch (SpeechRecognitionException e) {
                e.printStackTrace();
            }
        }
        @Override
        @WorkerThread
        public void onInactive() {
            Message msg = Message.obtain(mHandler, R.id.clientInactive);
            msg.sendToTarget();
        }
        @Override
        @WorkerThread
        public void onReady() {
            Message msg = Message.obtain(mHandler, R.id.clientReady);
            msg.sendToTarget();
        }
        @Override
        @WorkerThread
        public void onRecord(short[] speech) {
            Message msg = Message.obtain(mHandler, R.id.audioRecording, speech);
            msg.sendToTarget();
        }
        @Override
        @WorkerThread
        public void onPartialResult(String result) {
            Message msg = Message.obtain(mHandler, R.id.partialResult, result);
            msg.sendToTarget();
        }
        @Override
        @WorkerThread
        public void onEndPointDetected() {
            Log.d(TAG, "Event occurred : EndPointDetected");
        }
        @Override
        @WorkerThread
        public void onResult(SpeechRecognitionResult result) {
            Message msg = Message.obtain(mHandler, R.id.finalResult, result);
            msg.sendToTarget();
        }
        @Override
        @WorkerThread
        public void onError(int errorCode) {
            Message msg = Message.obtain(mHandler, R.id.recognitionError, errorCode);
            msg.sendToTarget();
        }
        @Override
        @WorkerThread
        public void onEndPointDetectTypeSelected(EndPointDetectType epdType) {
            Message msg = Message.obtain(mHandler, R.id.endPointDetectTypeSelected, epdType);
            msg.sendToTarget();
        }
    }

</code>
******************
CSS
==================

### CSS API란?
------------------
Clova Speech Synthesis API는 음성으로 변환할 텍스트를 입력받은 후 파라미터로 지정된 음색과 속도로 음성을 합성하여 그 결과를 되돌려준다. 이는 HTTP(하이퍼 미디어 환경에서 빠르고 간편하게 데이터를 전송하는 프로토콜) 기반의 REST API 이며, 사용자 인증인 로그인이 필요하지 않은 비로그인 Open API 이다.

### 사전 준비사항
-------------------
    1. 사용 API에서 CSS API 선택.
    2. 개발하려는 애플리케이션을 네이버 개발자 센터에 등록.
    3. 등록 시 API에 대한 권한 설정 및 필요한 인증 정보 획득.
    4. 등록 후 내 애플리케이션 메뉴로 이동 후 등록한 앱의 정보 확인.
        (Client ID, Client Secret)
   