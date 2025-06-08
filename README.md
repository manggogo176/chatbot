# 챗봇

## 레벤슈타인 거리를 이용한 챗봇 구하기


- pandas 설치

```
pip install pandas

```

- 챗봇 구현

- chatbot.py 

```
import pandas as pd

# 레벤슈타인 거리를 계산하는 함수입니다.
def calc_levenshtein_distance(a, b):
    """
    두 문자열 a와 b 사이의 레벤슈타인 거리를 계산합니다.
        """
    # 두 문자열이 같으면 거리는 0 입니다.
    if a == b:
        return 0
    
    # a와 b의 길이를 저장합니다
    a_len, b_len = len(a), len(b)

    # a 또는 b가 빈 문자열이면, 다른 문자열의 길이가 거리입니다.
    if a == "":
        return b_len
    if b == "":
        return a_len

    # 동적 프로그래밍을 위한 2차원 행렬(matrix) 초기화합니다.
    # (a_len + 1) x (b_len + 1) 크기의 행렬을 생성하고 0으로 채웁니다.
    matrix = [[0 for _ in range(b_len + 1)] for _ in range(a_len + 1)]

    # 행렬의 첫 번째 행과 열을 초기화합니다.
    for i in range(a_len + 1):
        matrix[i][0] = i
    for j in range(b_len + 1):
        matrix[0][j] = j

    # 행렬의 나머지 부분을 채워나감
    for i in range(1, a_len + 1):
        for j in range(1, b_len + 1):
            # 문자가 같으면 비용은 0, 다르면 1
            cost = 0 if a[i - 1] == b[j - 1] else 1
            
            # 삽입, 삭제, 치환 중 가장 작은 비용을 선택하는 부분입니다.
            matrix[i][j] = min(
                matrix[i - 1][j] + 1,        # 삭제 (Deletion)
                matrix[i][j - 1] + 1,        # 삽입 (Insertion)
                matrix[i - 1][j - 1] + cost  # 치환 (Substitution)
            )

    # 최종적으로 계산된 거리를 반환 (행렬의 가장 오른쪽 아래 값)
    return matrix[a_len][b_len]


class LevenshteinChatBot:
    # 챗봇 객체 초기화
    def __init__(self, filepath):
        # 데이터 파일을 로드하여 질문과 답변 리스트를 멤버 변수로 저장합니다.
        self.questions, self.answers = self.load_data(filepath)

    # CSV 파일로부터 질문과 답변 데이터를 불러오는 메서드입니다.
    def load_data(self, filepath):
        # ChatbotData.csv 파일을 읽어옵니다.
        data = pd.read_csv(filepath)
        # 'Q' 열을 리스트로 변환하여 questions에 저장
        questions = data['Q'].tolist()
        # 'A' 열을 리스트로 변환하여 answers에 저장
        answers = data['A'].tolist()
        return questions, answers

    # 입력 문장과 가장 유사한 질문을 찾아 답변을 반환하는 메서드입니다.
    def find_best_answer(self, input_sentence):
        # 가장 작은 레벤슈타인 거리를 저장할 변수, 초기값은 큰 수로 설정
        min_distance = float('inf')
        # 가장 유사한 질문의 인덱스를 저장할 변수
        best_match_index = -1

        # 학습 데이터의 모든 질문에 대해 반복
        for i, question in enumerate(self.questions):
            # 입력 문장과 현재 질문 사이의 레벤슈타인 거리를 계산
            distance = calc_levenshtein_distance(input_sentence, question)

            # 계산된 거리가 현재까지의 최소 거리보다 작으면
            if distance < min_distance:
                # 최소 거리를 현재 거리로 업데이트
                min_distance = distance
                # 최고 일치 인덱스를 현재 인덱스로 업데이트
                best_match_index = i
        
        # 가장 유사한 질문에 해당하는 답변을 반환합니다.
        return self.answers[best_match_index]

# - 챗봇 실행 부분 -

# 데이터 파일 경로 지정
filepath = 'ChatbotData.csv'

# 레벤슈타인 챗봇 객체 생성
chatbot = LevenshteinChatBot(filepath)

print("챗봇을 시작합니다. '종료'를 입력하면 대화를 마칩니다.")

# '종료'가 입력될 때까지 무한 루프 실행
while True:
    # 사용자로부터 질문 입력받기
    input_sentence = input('You: ')
    
    # 사용자가 '종료'를 입력하면 루프 종료
    if input_sentence.lower() == '종료':
        print('Chatbot: 안녕히 가세요.')
        break
        
    # 입력된 질문에 가장 적절한 답변 찾기
    response = chatbot.find_best_answer(input_sentence)
    
    # 챗봇의 답변 출력
    print('Chatbot:', response)

```