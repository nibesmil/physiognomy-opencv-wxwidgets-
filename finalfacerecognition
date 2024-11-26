//얼굴을 인식받아 얼굴형, 눈, 코, 입 수치 분석 및 관상 출력 프로그램

#include <wx/wx.h>
#include <wx/sizer.h>
#include <wx/statbmp.h>
#include <opencv2/opencv.hpp>
#include <opencv2/objdetect.hpp>
#include <iostream>
#include <thread>
#include <numeric>
#include <vector>

using namespace cv;
using namespace std;

// 메인 프레임 클래스 정의
class MainFrame : public wxFrame {
public:
    MainFrame(const wxString& title);

private:
    wxBitmapButton* startButton;    // 얼굴 인식 버튼
    wxBitmapButton* exitButton;    // 종료 버튼
    wxBitmapButton* saveButton;

    void OnStart(wxCommandEvent& event);
    void OnExit(wxCommandEvent& event);
    void OnSave(wxCommandEvent& event);

    void RunFaceDetection(string windowTitle);  // 얼굴 인식 실행 함수

    wxDECLARE_EVENT_TABLE();
};

wxBEGIN_EVENT_TABLE(MainFrame, wxFrame)
EVT_BUTTON(1001, MainFrame::OnStart)
EVT_BUTTON(1005, MainFrame::OnSave)
EVT_BUTTON(1003, MainFrame::OnExit)

wxEND_EVENT_TABLE()

// 메인 프레임 생성자
MainFrame::MainFrame(const wxString& title)
    : wxFrame(NULL, wxID_ANY, title, wxDefaultPosition, wxSize(800, 600)) {

    // 아이콘 설정 (아이콘 파일 경로를 설정)
    wxIcon icon;
    icon.LoadFile("iconzz.ico", wxBITMAP_TYPE_ICO); // 아이콘 파일 경로 설정
    SetIcon(icon);  // 윈도우 아이콘으로 설정

    // 배경색 설정
    SetBackgroundColour(*wxBLACK);

    // 레이아웃 설정
    wxBoxSizer* mainSizer = new wxBoxSizer(wxVERTICAL);

    wxStaticText* logoText = new wxStaticText(this, wxID_ANY, wxT("관觀\n상相"));
    logoText->SetPosition(wxPoint(10, 10));  // (10, 10) 위치에 배치
    wxFont logoFont(50, wxFONTFAMILY_ROMAN, wxFONTSTYLE_NORMAL, wxFONTWEIGHT_BOLD, false, wxT("궁서"));
    logoText->SetFont(logoFont);
    logoText->SetForegroundColour(*wxWHITE); // 텍스트 색상 흰색으로 설정

    wxImage::AddHandler(new wxPNGHandler()); // PNG 파일 지원 핸들러 추가
    // 버튼 이미지 생성 및 리사이즈
    wxImage startImage(wxT("cam.png"), wxBITMAP_TYPE_PNG);
    wxImage exitImage(wxT("exit.png"), wxBITMAP_TYPE_PNG);
    wxImage saveImage(wxT("save.png"), wxBITMAP_TYPE_PNG);

    startImage.Rescale(100, 100);
    exitImage.Rescale(100, 100);
    saveImage.Rescale(100, 100);

    // 버튼 생성
    startButton = new wxBitmapButton(this, 1001, wxBitmap(startImage), wxDefaultPosition, wxSize(100, 100));
    saveButton = new wxBitmapButton(this, 1005, wxBitmap(saveImage), wxDefaultPosition, wxSize(100, 100));
    exitButton = new wxBitmapButton(this, 1003, wxBitmap(exitImage), wxDefaultPosition, wxSize(100, 100));

    // 텍스트 추가
    wxStaticText* startText = new wxStaticText(this, wxID_ANY, wxT("Start"), wxDefaultPosition, wxDefaultSize, wxALIGN_CENTER);
    wxStaticText* exitText = new wxStaticText(this, wxID_ANY, wxT("Exit"), wxDefaultPosition, wxDefaultSize, wxALIGN_CENTER);
    wxStaticText* saveText = new wxStaticText(this, wxID_ANY, wxT("저장 내역"), wxDefaultPosition, wxDefaultSize, wxALIGN_CENTER);

    // 폰트 설정
    wxFont textFont(10, wxFONTFAMILY_ROMAN, wxFONTSTYLE_NORMAL, wxFONTWEIGHT_NORMAL, false, wxT("맑은 고딕"));
    startText->SetFont(textFont);
    exitText->SetFont(textFont);
    saveText->SetFont(textFont);

    // 텍스트 색상 설정 (흰색)
    startText->SetForegroundColour(*wxWHITE);
    exitText->SetForegroundColour(*wxWHITE);
    saveText->SetForegroundColour(*wxWHITE);

    // 버튼과 텍스트를 묶을 BoxSizer 생성
    wxBoxSizer* startSizer = new wxBoxSizer(wxVERTICAL);
    startSizer->Add(startButton, 0, wxALIGN_CENTER);
    startSizer->Add(startText, 0, wxALIGN_CENTER | wxTOP, 5);

    wxBoxSizer* exitSizer = new wxBoxSizer(wxVERTICAL);
    exitSizer->Add(exitButton, 0, wxALIGN_CENTER);
    exitSizer->Add(exitText, 0, wxALIGN_CENTER | wxTOP, 5);

    wxBoxSizer* saveSizer = new wxBoxSizer(wxVERTICAL);
    saveSizer->Add(saveButton, 0, wxALIGN_CENTER);
    saveSizer->Add(saveText, 0, wxALIGN_CENTER | wxTOP, 5);

    // 버튼과 텍스트를 포함하는 Sizer를 설정
    wxGridSizer* buttonSizer = new wxGridSizer(1, 3, 10, 10); // 1행 3열로 설정
    buttonSizer->Add(startSizer, 0, wxALIGN_CENTER);  // Start 버튼
    buttonSizer->Add(saveSizer, 0, wxALIGN_CENTER);   // 저장 내역 버튼
    buttonSizer->Add(exitSizer, 0, wxALIGN_CENTER);   // Exit 버튼


    mainSizer->Add(logoText, 0, wxALIGN_CENTER | wxTOP, 20);   // 텍스트 상단에 배치
    mainSizer->AddStretchSpacer(1);                           // 로고와 버튼 사이의 여백 추가
    mainSizer->Add(buttonSizer, 0, wxALIGN_CENTER | wxALL, 20); // 버튼을 한 줄에 배치
    mainSizer->AddStretchSpacer(1);                           // 버튼 하단 여유 공간 추가


    SetSizer(mainSizer);
    Centre(); // 화면 중앙에 배치
}
void MainFrame::OnStart(wxCommandEvent& event) {
    // 사용자 이름을 입력받는 다이얼로그 생성
    wxTextEntryDialog nameDialog(this, wxT("사용자 이름을 입력해주세요:"), wxT("이름 입력"), wxEmptyString);

    if (nameDialog.ShowModal() == wxID_OK) {
        wxString userName = nameDialog.GetValue();  // 사용자가 입력한 이름

        if (userName.IsEmpty()) {
            wxMessageBox("이름을 입력해야 합니다.", "오류", wxOK | wxICON_ERROR);
            return;
        }

        // 입력받은 이름을 사용하여 웹캠 창 이름 설정
        string windowTitle = wxString(userName + " ) 얼굴 인식 중 . . .").ToStdString();

        // 메시지박스를 띄워 사용자에게 안내
        wxMessageBox("얼굴 인식을 시작합니다.\n얼굴 정면이 잘 보이도록 카메라를 세팅해주세요 !", "I N F O", wxOK | wxICON_INFORMATION);

        this_thread::sleep_for(std::chrono::seconds(2)); // 잠시 대기

        // 입력받은 이름을 사용해 얼굴 인식 실행
        thread(&MainFrame::RunFaceDetection, this, windowTitle).detach();
    }
}

void MainFrame::OnExit(wxCommandEvent& event) {
    Close(true);
}

void MainFrame::OnSave(wxCommandEvent& event) {
    Close(true);
}

string DetermineFaceShape(double faceWidth, double faceHeight, double jawWidth) {
    double aspectRatio = faceHeight / faceWidth; // 얼굴 높이/너비 비율
    double jawToFaceWidthRatio = jawWidth / faceWidth; // 턱 너비/얼굴 너비 비율

    // 조정된 기준으로 얼굴형 판단
    if (aspectRatio > 1.5) {
        return "Long";
    }
    else if (aspectRatio < 1.1 && jawToFaceWidthRatio > 0.85) {
        return "Round"; // 동그란 얼굴형을 더 정확히 감지
    }
    else if (jawToFaceWidthRatio < 0.65) {
        return "Oval";
    }
    else if (jawToFaceWidthRatio > 0.95) {
        return "Square";
    }
    else {
        return "Diamond";
    }
}

void MainFrame::RunFaceDetection(string windowTitle) {
    CascadeClassifier faceCascade, eyeCascade, mouthCascade, noseCascade;

    if (!faceCascade.load("haarcascade_frontalface_default.xml") ||
        !eyeCascade.load("haarcascade_eye.xml") ||
        !mouthCascade.load("haarcascade_mcs_mouth.xml") ||
        !noseCascade.load("haarcascade_mcs_nose.xml")) {
        wxMessageBox("Cascade 파일을 로드할 수 없습니다. 파일 경로를 확인하세요.", "E R R O R !", wxOK | wxICON_ERROR);
        return;
    }

    VideoCapture cap(0);
    if (!cap.isOpened()) {
        wxMessageBox("웹캠을 실행할 수 없습니다. 설정을 확인하세요.", "E R R O R !", wxOK | wxICON_ERROR);
        return;
    }

    Mat frame;
    bool faceDetected = false;
    double startTime = 0.0;

    vector<int> eyeWidths, eyeHeights, noseWidths, noseHeights, mouthWidths, mouthHeights;

    auto calc_average = [](const vector<int>& values) {
        return values.empty() ? 0.0 : accumulate(values.begin(), values.end(), 0.0) / values.size();
        };

    string faceShape;

    while (cap.read(frame)) {
        vector<Rect> faces;
        faceCascade.detectMultiScale(frame, faces, 1.1, 4, 0, Size(30, 30));

        for (const auto& face : faces) {
            rectangle(frame, face, Scalar(255, 0, 0), 2);

            double faceWidth = face.width;
            double faceHeight = face.height;
            double jawWidth = face.width * 0.8;

            faceShape = DetermineFaceShape(faceWidth, faceHeight, jawWidth);
            putText(frame, faceShape, Point(face.x, face.y - 10), FONT_HERSHEY_SIMPLEX, 0.7, Scalar(0, 255, 255), 2);

            Mat faceROI = frame(face);
            vector<Rect> eyes, mouths, noses;

            eyeCascade.detectMultiScale(faceROI, eyes);
            mouthCascade.detectMultiScale(faceROI, mouths);
            noseCascade.detectMultiScale(faceROI, noses);

            for (const auto& eye : eyes) {
                eyeWidths.push_back(eye.width);
                eyeHeights.push_back(eye.height);
                rectangle(frame, eye + Point(face.x, face.y), Scalar(0, 255, 0), 2);
            }
            for (const auto& mouth : mouths) {
                mouthWidths.push_back(mouth.width);
                mouthHeights.push_back(mouth.height);
                rectangle(frame, mouth + Point(face.x, face.y), Scalar(0, 0, 255), 2);
            }
            for (const auto& nose : noses) {
                noseWidths.push_back(nose.width);
                noseHeights.push_back(nose.height);
                rectangle(frame, nose + Point(face.x, face.y), Scalar(255, 255, 0), 2);
            }

            if (!faceDetected) {
                startTime = static_cast<double>(getTickCount());
                faceDetected = true;
            }
        }

        double avgEyeWidth = calc_average(eyeWidths);
        double avgEyeHeight = calc_average(eyeHeights);
        double avgNoseWidth = calc_average(noseWidths);
        double avgNoseHeight = calc_average(noseHeights);
        double avgMouthWidth = calc_average(mouthWidths);
        double avgMouthHeight = calc_average(mouthHeights);

        // 실시간 분석 결과를 웹캠 화면에 출력
        stringstream avgText;
        avgText << "eyeavg : " << avgEyeWidth << "x" << avgEyeHeight;
        putText(frame, avgText.str(), Point(10, 50), FONT_HERSHEY_SIMPLEX, 0.5, Scalar(0, 0, 0), 2);

        avgText.str("");
        avgText.clear();
        avgText << "noseavg : " << avgNoseWidth << "x" << avgNoseHeight;
        putText(frame, avgText.str(), Point(10, 80), FONT_HERSHEY_SIMPLEX, 0.5, Scalar(0, 0, 0), 2);

        avgText.str("");
        avgText.clear();
        avgText << "mouthavg : " << avgMouthWidth << "x" << avgMouthHeight;
        putText(frame, avgText.str(), Point(10, 110), FONT_HERSHEY_SIMPLEX, 0.5, Scalar(0, 0, 0), 2);

        if (faceDetected) {
            double elapsedTime = (static_cast<double>(getTickCount()) - startTime) / getTickFrequency();
            if (elapsedTime >= 5.0) {
                wxMessageBox("얼굴, 눈, 코, 입 수치를 통해 관상을 추출중입니다.\n잠시만 기다려 주세요.", "I N F O", wxOK | wxICON_INFORMATION);

                wxFrame* emptyFrame = new wxFrame(nullptr, wxID_ANY, "얼굴 분석 통한 관상 추출 결과", wxDefaultPosition, wxSize(800, 600));
                emptyFrame->SetBackgroundColour(wxColour(255, 255, 255));

                // wxPanel 생성
                wxPanel* panel = new wxPanel(emptyFrame);

                // 궁서체 설정
                wxFont titleFont(wxFontInfo(14).Family(wxFONTFAMILY_ROMAN).FaceName("궁서"));
                wxFont resultFont(wxFontInfo(12).Family(wxFONTFAMILY_ROMAN).FaceName("궁서"));
                wxFont conclusionFont(wxFontInfo(14).Family(wxFONTFAMILY_ROMAN).FaceName("궁서"));

                // 결과 제목 추가
                wxStaticText* titleText = new wxStaticText(panel, wxID_ANY, "<얼굴 분석 결과>", wxPoint(0, 20), wxSize(760, 50), wxALIGN_CENTER);
                titleText->SetForegroundColour(wxColour(0, 0, 0)); // 텍스트 색상 설정
                titleText->SetFont(titleFont); // 궁서체 폰트 설정

                // 분석 결과를 위한 stringstream
                stringstream analysisText;
                analysisText << "얼굴 형태: " << faceShape << "\n";
                analysisText << "평균 눈 크기: " << avgEyeWidth << " x " << avgEyeHeight << "\n";
                analysisText << "평균 코 크기: " << avgNoseWidth << " x " << avgNoseHeight << "\n";
                analysisText << "평균 입 크기: " << avgMouthWidth << " x " << avgMouthHeight << "\n";

                // 분석 결과 텍스트 추가
                wxStaticText* resultText = new wxStaticText(panel, wxID_ANY, analysisText.str(), wxPoint(0, 80), wxSize(760, 150), wxALIGN_CENTER);
                resultText->SetForegroundColour(wxColour(0, 0, 0)); // 텍스트 색상 설정
                resultText->SetFont(resultFont); // 궁서체 폰트 설정

                // 관상 출력 결과 제목 추가
                wxStaticText* conclusionText = new wxStaticText(panel, wxID_ANY, "<관상 출력 결과>", wxPoint(0, 250), wxSize(760, 50), wxALIGN_CENTER);
                conclusionText->SetForegroundColour(wxColour(0, 0, 0)); // 텍스트 색상 설정
                conclusionText->SetFont(conclusionFont); // 궁서체 폰트 설정

                // 간단한 관상 판단을 위한 예시
                string fortuneResult = "관상이 좋습니다."; // 기본 값

                // 얼굴형에 따른 기본 관상
                if (faceShape == "Round") {
                    fortuneResult = "부드럽고 친근합니다.\n동그란 얼굴형은 일반적으로 사람들에게 친근감을 주며, 화목한 가정을 이룰 수 있는 가능성이 큽니다.";
                }
                else if (faceShape == "Long") {
                    fortuneResult = "강한 인상을 줍니다.\n길쭉한 얼굴형은 대개 리더십이나 자기주장이 강한 성격을 나타내며, 때때로 고집이 세기도 합니다.";
                }
                else if (faceShape == "Oval") {
                    fortuneResult = "균형 잡히고 온화합니다.\n타고난 아름다움을 갖추고 있으며, 전체적인 조화와 균형이 좋은 사람에게 나타납니다.";
                }
                else if (faceShape == "Square") {
                    fortuneResult = "강한 의지를 나타냅니다.\n사각형 얼굴형은 자존감이 높고, 사람들에게 신뢰감을 주는 성격을 나타냅니다.";
                }
                else if (faceShape == "Diamond") {
                    fortuneResult = "창의적이고 개성 있는 인상을 줍니다.\n다이아몬드형 얼굴은 대개 독창적이고 예술적인 성향을 가진 사람에게 나타납니다.";
                }

                // 얼굴 부위에 따른 판단
                if (avgEyeWidth < 40 || avgEyeHeight < 30) {
                    fortuneResult += "\n\n눈이 너무 작아 불운할 수 있습니다.\n눈이 작은 사람은 대체로 신뢰를 얻는 데 어려움을 겪을 수 있으며, 감정 표현이 부족할 수 있습니다.";
                }
                else if (avgEyeWidth > 60) {
                    fortuneResult += "\n\n눈이 크고 매력적입니다.\n눈이 큰 사람은 대개 사람들과의 소통이 뛰어나며, 감정을 잘 표현하고 리더십을 발휘할 수 있습니다.";
                }

                if (avgMouthWidth < 40) {
                    fortuneResult += "\n\n입이 좁아 사람들과의 교류가 어렵습니다.\n입이 좁은 사람은 때때로 말수가 적고, 감정을 표현하는 데 어려움을 겪을 수 있습니다.";
                }
                else if (avgMouthWidth > 60) {
                    fortuneResult += "\n\n입이 넓어 사교성이 뛰어나고, 타인과의 관계에서 좋은 영향을 끼칠 수 있습니다.\n다만, 과도한 표현은 종종 오해를 살 수 있습니다.";
                }

                if (avgNoseHeight < 40) {
                    fortuneResult += "\n\n코가 작은 사람은 주변 사람들에게 신뢰를 주는 데 어려움을 겪을 수 있습니다.\n그러나 작은 코는 아담하고 귀여운 인상을 줄 수 있습니다.";
                }
                else if (avgNoseHeight > 60) {
                    fortuneResult += "\n\n코가 크고 뚜렷한 사람은 대개 자기 주장이 강하고, 타인의 관심을 끌기 쉬운 성격을 가지고 있습니다.";
                }

                // 눈, 코, 입의 비율에 따른 분석
                double totalFacialRatio = (avgEyeWidth + avgNoseHeight + avgMouthWidth) / 3.0;
                if (totalFacialRatio > 70) {
                    fortuneResult += "\n\n얼굴 비율이 균형 잡혀 있어 조화로운 외모를 가지고 있습니다.\n전체적인 얼굴 비율이 좋으면, 대체로 사회적인 성공과 좋은 인간관계를 유지할 가능성이 큽니다.";
                }
                else if (totalFacialRatio < 50) {
                    fortuneResult += "\n\n얼굴 비율에 균형이 부족해 다소 불안정한 인상을 줄 수 있습니다.\n그러나 잘 다듬어진 스타일링이나 관리로 이를 보완할 수 있습니다.";
                }

                // 전체적인 성격 분석
                if (avgEyeWidth > 60 && avgMouthWidth > 60) {
                    fortuneResult += "\n\n전체적으로 자신감이 넘치는 성격을 지니고 있으며,\n사람들과의 관계에서 자연스럽고 활발한 태도를 보일 것입니다.";
                }
                else if (avgEyeWidth < 40 && avgMouthWidth < 40) {
                    fortuneResult += "\n\n조용하고 내성적인 성격을 가진 경우가 많습니다.\n그러나 때때로 이로 인해 사람들과의 교류가 어려울 수 있습니다.";
                }

                // 결과 텍스트 추가
                wxStaticText* conclusionResultText = new wxStaticText(panel, wxID_ANY, fortuneResult, wxPoint(0, 300), wxSize(760, 100), wxALIGN_CENTER);
                conclusionResultText->SetForegroundColour(wxColour(0, 0, 0)); // 텍스트 색상 설정
                conclusionResultText->SetFont(resultFont); // 궁서체 폰트 설정


                // 프레임 표시
                emptyFrame->Show(true);
                cap.release();
                waitKey(0);
                destroyAllWindows();
                break;
            }
        }

        imshow(windowTitle, frame);
        if (waitKey(1) == 27) break;
    }

    destroyAllWindows();
}

class MyApp : public wxApp {
public:
    virtual bool OnInit();
};

IMPLEMENT_APP(MyApp)

bool MyApp::OnInit() {
    MainFrame* frame = new MainFrame("觀 相.");
    frame->Show(true);
    return true;
}
