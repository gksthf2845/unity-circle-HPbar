using UnityEngine;
using UnityEngine.UI; // MaskableGraphic을 사용하기 위해 필요합니다.
using System.Collections.Generic;

public class hpBar : MaskableGraphic
{
   
    // Inspector에서 0.0에서 1.0 사이의 슬라이더로 표시됩니다.
    // 슬라이더 범위 추가
    public float healthRatio = 1f; // 0.0 (empty) to 1.0 (full)

    public float thickness = 10f; // 게이지 두께
   
    // 최소 3개의 세그먼트 (삼각형)
    // 최소 세그먼트 수 제한
    public int segments = 100; // 원형 게이지를 구성할 세그먼트 수 (높을수록 부드러움)

    public Gradient healthGradient; // HP에 따른 색상 그라디언트

    // 게이지 시작점 (예: Top, Right, Bottom, Left)
    public enum FillOrigin { Bottom, Left, Top, Right }
    public FillOrigin fillOrigin = FillOrigin.Top;
    // 시계 방향 또는 반시계 방향
    public bool clockwise = true;

    // HealthRatio 프로퍼티를 통해 값을 변경할 때 SetVerticesDirty()를 호출하여 메시를 업데이트합니다.
    // 스크립트에서 값을 변경할 때도 SetVerticesDirty()가 호출되도록 프로퍼티를 사용합니다.[1, 2]
    public float HealthRatio
    {
        get { return healthRatio; }
        set
        {
            healthRatio = Mathf.Clamp01(value);
            SetVerticesDirty(); // 메시 업데이트를 요청합니다.[1, 2]
        }
    }

    // RectTransform의 크기가 변경될 때 메시를 다시 그리도록 합니다.
    // rectTransform은 MaskableGraphic의 부모 클래스인 Graphic에서 상속받는 속성입니다.[3]
    // 만약 rectTransform에 NaN 오류가 발생한다면, Canvas 설정이나 부모 레이아웃 컴포넌트(예: Horizontal/Vertical Layout Group)를 확인해야 합니다.
    protected override void OnRectTransformDimensionsChange()
    {
        base.OnRectTransformDimensionsChange();
        SetVerticesDirty();
    }

    // OnValidate는 에디터에서 값이 변경될 때 호출되어 즉시 업데이트를 반영합니다.
    #if UNITY_EDITOR
    protected override void OnValidate()
    {
        base.OnValidate();
        SetVerticesDirty();
    }
    #endif

    protected override void OnPopulateMesh(VertexHelper vh)
    {
        vh.Clear(); // 기존 메시를 지웁니다.[4]

        // RectTransform의 크기가 유효한지 확인합니다.
        // rectTransform.rect.width 또는 height가 0이거나 NaN이면 그리지 않습니다.
        if (rectTransform.rect.width <= 0 || rectTransform.rect.height <= 0 || segments == 0)
        {
            return;
        }

        float outerRadius = rectTransform.rect.width / 2f;
        float innerRadius = outerRadius - thickness;

        // 내부 반지름이 0보다 작거나 같으면 그리지 않습니다.
        if (innerRadius < 0) // 0이 아닌 음수일 경우만 처리
        {
            innerRadius = 0; // 최소값으로 설정하여 오류 방지
        }
        // 만약 thickness가 outerRadius보다 커서 innerRadius가 음수가 되면,
        // 게이지가 뒤집히거나 이상하게 보일 수 있으므로, 이 경우도 방지합니다.
        if (thickness > outerRadius)
        {
            thickness = outerRadius; // 두께를 최대 외부 반지름으로 제한
            innerRadius = 0;
        }


        // 피벗을 고려한 중심점 계산
        Vector2 center = rectTransform.pivot * rectTransform.rect.size; 

        // 전체 바의 색상을 healthRatio에 따라 결정합니다.
        // HP가 줄어들수록 색상이 붉어지도록 그라디언트의 전체 범위를 healthRatio에 매핑합니다.
        Color barColor = healthGradient.Evaluate(healthRatio); 

        float startAngleOffset = GetStartAngleOffset(fillOrigin); // 시작 각도 오프셋 계산

        float currentFillAngle = healthRatio * 360f; // 현재 채워진 각도 (0~360도)

        // 메인 아크 그리기
        DrawArc(vh, center, innerRadius, outerRadius, startAngleOffset, currentFillAngle, barColor, segments, clockwise);

        // 둥근 끝 그리기
        // 시작점 둥근 끝
        DrawRoundedCap(vh, center, outerRadius, innerRadius, startAngleOffset, barColor, true, clockwise);
        // 끝점 둥글 끝
        // 끝점의 각도는 시작 각도에서 채워진 각도만큼 진행한 위치입니다.
        float endAngle = startAngleOffset + (clockwise? -currentFillAngle : currentFillAngle);
        DrawRoundedCap(vh, center, outerRadius, innerRadius, endAngle, barColor, false, clockwise);
    }

    // 아크를 그리는 헬퍼 함수
    private void DrawArc(VertexHelper vh, Vector2 center, float innerRadius, float outerRadius, float startAngle, float sweepAngle, Color color, int segments, bool clockwise)
    {
        int currentVertCount = vh.currentVertCount; // Capture current vertex count before adding arc vertices
        float angleStep = sweepAngle / segments;

        for (int i = 0; i <= segments; i++)
        {
            float angle = startAngle + (clockwise? -angleStep * i : angleStep * i);
            float rad = angle * Mathf.Deg2Rad;

            Vector2 outerVertex = center + new Vector2(Mathf.Cos(rad) * outerRadius, Mathf.Sin(rad) * outerRadius);
            Vector2 innerVertex = center + new Vector2(Mathf.Cos(rad) * innerRadius, Mathf.Sin(rad) * innerRadius);

            vh.AddVert(outerVertex, color, Vector2.zero); // UV는 필요 없으므로 Vector2.zero
            vh.AddVert(innerVertex, color, Vector2.zero);

            if (i > 0)
            {
                int prevOuter = currentVertCount + (i - 1) * 2;
                int prevInner = currentVertCount + (i - 1) * 2 + 1;
                int currOuter = currentVertCount + i * 2;
                int currInner = i * 2 + 1; // 현재 추가된 정점의 인덱스 (vh.currentVertCount - 1)

                // vh.AddVert 호출 후 vh.currentVertCount가 증가하므로,
                // 현재 추가된 정점의 인덱스는 vh.currentVertCount - 2 (outer)와 vh.currentVertCount - 1 (inner)입니다.
                // 따라서 currOuter와 currInner는 루프 내에서 계산된 i를 기반으로 해야 합니다.
                // 이전에 추가된 정점의 인덱스를 기준으로 삼각형을 구성합니다.
                vh.AddTriangle(prevOuter, currOuter, currInner);
                vh.AddTriangle(currInner, prevInner, prevOuter);
            }
        }
    }

    // 둥근 끝을 그리는 헬퍼 함수
    private void DrawRoundedCap(VertexHelper vh, Vector2 center, float outerRadius, float innerRadius, float angle, Color color, bool isStartCap, bool clockwise)
    {
        int capSegments = 32; // 더 부드러운 곡선을 위해 세그먼트 수 증가
        float thickness = outerRadius - innerRadius;
        float capRadius = thickness / 2f;

        // 방향 벡터 계산
        float rad = angle * Mathf.Deg2Rad;
        Vector2 directionVector = new Vector2(Mathf.Cos(rad), Mathf.Sin(rad));
        
        // 캡의 중심점 계산
        float midRadius = (outerRadius + innerRadius) / 2f;
        Vector2 capCenter = center + directionVector * midRadius;

        // 반원을 그리기 위한 각도 계산
        float halfCircleStart = angle + (isStartCap ? 180f : 0f);
        float sweepAngle = 180f;

        List<Vector2> vertices = new List<Vector2>();

        // 시작점과 끝점 추가 (외부와 내부 연결점)
        Vector2 outerPoint = center + directionVector * outerRadius;
        Vector2 innerPoint = center + directionVector * innerRadius;
        vertices.Add(outerPoint);
        vertices.Add(innerPoint);

        // 반원형 캡 생성
        for (int i = 0; i <= capSegments; i++)
        {
            float t = i / (float)capSegments;
            float currentAngle = halfCircleStart + sweepAngle * t;
            float currentRad = currentAngle * Mathf.Deg2Rad;
            
            Vector2 circlePoint = capCenter + new Vector2(
                Mathf.Cos(currentRad) * capRadius,
                Mathf.Sin(currentRad) * capRadius
            );
            
            vertices.Add(circlePoint);
        }

        // 정점 추가
        int baseIndex = vh.currentVertCount;
        for (int i = 0; i < vertices.Count; i++)
        {
            vh.AddVert(vertices[i], color, Vector2.zero);
        }

        // 삼각형 생성 - 부채꼴 형태로
        for (int i = 2; i < vertices.Count - 1; i++)
        {
            // 외부 점과 연결
            vh.AddTriangle(baseIndex, baseIndex + i, baseIndex + i + 1);
            // 내부 점과 연결
            vh.AddTriangle(baseIndex + 1, baseIndex + i + 1, baseIndex + i);
        }
    }

    // Image.FillOrigin에 따른 시작 각도 오프셋을 반환합니다.
    private float GetStartAngleOffset(FillOrigin origin)
    {
        switch (origin)
        {
            case FillOrigin.Bottom: return 90f;
            case FillOrigin.Left: return 180f;
            case FillOrigin.Top: return 270f; // 또는 -90f
            case FillOrigin.Right: return 0f;
            default: return 0f;
        }
    }
}
