

##  TanStack Query를 사용한 이유

- **오답 문제 불러오기 시 캐싱을 통한 성능 최적화**  
   → 동일한 요청에 대해 불필요한 API 호출을 방지하고, 캐시 데이터를 우선 활용하여 효율적 렌더링

- **데이터 변경 시점에만 쿼리 무효화(invalidate) 처리**  
   → 오답 문제 추가나 삭제 시에만 서버 데이터를 갱신하도록 하여, 네트워크 리소스 효율상승

- **isPending / error 등의 상태 관리 자동화**  
   → 로딩, 에러, 성공 상태를 코드로 직접 관리하지 않고도 상황별 UI를 간결하게 렌더링가능


---

## 오답문제 불러오기 (useQuery)

```jsx
  const getIncorrectAnswers = async () => {
    try {
      const response = await privateInstance.get('/api/incorrect-answers');
      return response.data;
    } catch (err) {
      console.log(err);
    }
  };
```

```jsx
import { useQuery } from '@tanstack/react-query';
import { useQuiz } from '../quizComp/hooks/useQuiz';
import type { IncorrectQuiz } from '../../types/quizTypes';
import AnswerHistoryCard from './IncorrectAnswers/AnswerHistoryCard';
import IncorrectModal from '../../components/IncorrectModal/IncorrectModal';
import { useState } from 'react';

const MyPageMain = () => {
  const { getIncorrectAnswers } = useQuiz();
  const [open, setOpen] = useState(false);
  const [selectedQuiz, setSelectedQuiz] = useState<IncorrectQuiz | null>(null);

  const { isPending, error, data } = useQuery<IncorrectQuiz[]>({
    queryKey: ['incorrectAnswer'],
    queryFn: getIncorrectAnswers,
    
  });

  const handleOpenModal = (quiz: IncorrectQuiz) => {
    setSelectedQuiz(quiz);
    setOpen(true);
  };

  const handleCloseModal = () => {
    setOpen(false);
    setSelectedQuiz(null);
  };

  if (isPending) return <>로딩중..</>;
   if (error) return <>에러발생: {error.message}</>;

  return (
    <div className='flex max-h-[50rem] overflow-auto flex-col gap-4 px-10'>
      {data?.map((item) => (
        <div key={item.id} onClick={() => handleOpenModal(item)}>
          <AnswerHistoryCard data={item} />
        </div>
      ))}

      {selectedQuiz && (
        <IncorrectModal
          data={selectedQuiz}
          isOpen={open}
          setIsopen={handleCloseModal}
        />
      )}
    </div>
  );
};

export default MyPageMain;

```


---

## 저장된 오답문제 삭제하기(useMuation)

```jsx
import { useQuizStore } from '../../../store/useQuizStore';
import { useOptionStore } from '../../../store/useOptionStore';
import { useUserStore } from '../../../store/useUserStore';
import { instance } from '../../../apis/instance';
import { privateInstance } from '../../../apis/privateInstance';
import { useState } from 'react';
import { useToastStore } from '../../../store/useToastStore';
import { useMutation, useQueryClient } from '@tanstack/react-query';


export const useQuiz = () => {
  const {
    result,
    quiz,
    setQuiz,
    userAnswer,
    setUserAnswer,
    setResult,
    setIsLoading,
    setIsGrading,
  } = useQuizStore();
  const { category, level } = useOptionStore();
  const { user } = useUserStore();
  const { addToast } = useToastStore();

  const queryClient = useQueryClient();

  const [error, setError] = useState<Error | null>(null);
  
  // 삭제로직을 제외한 나머지 로직들은 생략
  const handleDeleteIncorrect = useMutation({
    mutationFn: async (incorrectId: string) => {
      const response = await privateInstance.delete(
        '/api/delete-incorrect-answer',
        {
          data: {
            userId: user?.id,
            id: incorrectId,
          },
        },
      );
      return response.data;
    },
    onSuccess: () => {
      addToast('success', '오답이 삭제되었습니다.');
      // 쿼리 무효화
      queryClient.invalidateQueries({ queryKey: ['incorrectAnswer'] });
    },
    onError: () => {
      addToast('error', '오답삭제중 오류가 발생했습니다');
    },
  });



  return {
    handleDeleteIncorrect,
    error,
  };
};

```

```jsx
import Button from '../../../components/Button';
import type { IncorrectQuiz } from '../../../types/quizTypes';
import { cn } from '../../../utils/cn';
import { useQuiz } from '../../quizComp/hooks/useQuiz';

interface AnswerHistoryCardProps {
  data: IncorrectQuiz;
}

const AnswerHistoryCard = ({ data }: AnswerHistoryCardProps) => {
  let topicClassName = '';
  let levelClassName = '';

  const { handleDeleteIncorrect } = useQuiz();


  const handleDelete = () => {
    handleDeleteIncorrect.mutate(data.id);
  };

  if (data.topic?.includes('React')) {
    topicClassName = 'text-blue-800';
  } else if (data.topic?.includes('TypeScript')) {
    topicClassName = 'text-sky-300';
  } else if (data.topic?.includes('JavaScript')) {
    topicClassName = 'text-yellow-500';
  } else if (data.topic?.includes('CSS')) {
    topicClassName = 'text-pink-500';
  } else {
    topicClassName = 'text-gray-600';
  }

  if (data.level === '쉬움') {
    levelClassName += 'text-green-600';
  } else if (data.level === '보통') {
    levelClassName += 'text-yellow-500';
  } else {
    levelClassName += 'text-red-600';
  }

  return (
    <div className='w-full rounded-xl border border-gray-300 bg-white px-10 py-5 shadow-sm hover:animate-pulse hover:cursor-pointer'>
      <h3 className='text-[1.2rem] font-bold text-black md:text-[1.4rem]'>
        {data.question}
      </h3>

      <div className='mt-2 flex justify-between space-y-1 text-gray-700'>
        <div className='text-md mt-2 flex flex-col gap-3 font-bold text-gray-500'>
          <p className={cn('text-xl font-bold text-black', topicClassName)}>
            {data.topic}
          </p>
          <p className={cn(`text-xl font-bold`, levelClassName)}>
            {data.level}
          </p>
        </div>

        <div>
          <Button
            onClick={() => handleDelete()}
            className='w-[6rem] py-[1rem] md:w-[10rem]'
          >
            삭제
          </Button>
        </div>
      </div>
    </div>
  );
};

export default AnswerHistoryCard;

```