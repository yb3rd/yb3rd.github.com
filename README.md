# yb3rd.github.com
Blog
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
 
public enum CheckType
{
    None,
 
    /// <summary> 圆形 </summary>
    Circle,
 
    /// <summary> 三角形 </summary>
    Triangle,
 
    /// <summary> 扇形 </summary>
    Fanshaped,
 
    /// <summary> 矩形 </summary>
    Rectangle,
 
    /// <summary> 扇面 </summary>
    Sector,
 
    /// <summary> 环形 </summary>
    Ring,
}
 
[ExecuteInEditMode]
public class RangeCheckScript : MonoBehaviour
{
    public CheckType currType = CheckType.None;
    public Transform mPalyer;
    public Transform mTarget;
    public bool mCheckOpen = true;
 
    void Update()
    {
        if (!mCheckOpen)
            return;
        if (null != mPalyer)
            Debug.DrawLine(mPalyer.position, mPalyer.position + mPalyer.forward * 8,Color.yellow);
 
 
        bool bResult = false;
        switch (currType)
        {
            case CheckType.None:
                break;
            case CheckType.Circle:
                bResult = CircleCheck(mPalyer, mTarget, 6);
                break;
            case CheckType.Triangle:
                bResult = TriangleCheck(mPalyer, mTarget, 1, 10);
                break;
            case CheckType.Fanshaped:
                bResult = FanshapedCheck(mPalyer, mTarget, 45, 5);
                break;
            case CheckType.Rectangle:
                //bResult = SimulateRectangleCheck(mPalyer, mTarget, 2, 8);
                bResult = RectangleCheck(mPalyer, mTarget, 2, 8);
                break;
            case CheckType.Sector:
                bResult = SectorCheck(mPalyer, mTarget, 45, 5, 8);
                break;
            case CheckType.Ring:
                bResult = RingCheck(mPalyer, mTarget, 4, 8);
                break;
            default:
                break;
        }
 
        if (bResult)
            Debug.LogError("检测到目标");
    }
 
    /// <summary>
    /// 圆形范围检测
    /// </summary>
    private bool CircleCheck(Transform self, Transform target, float distance)
    {
        if (null == self || null == target)
            return false;
 
        //---------------------绘制图形-----------------------------------
 
        Vector3 selfPosition = NormalizePosition(self.position);
        Vector3 targetPosition = NormalizePosition(target.position);
 
        int nCircleDentity = 360;
        Vector3 beginPoint = selfPosition;
        Vector3 endPoint = Vector3.zero;
        float tempStep = 2 * Mathf.PI / nCircleDentity;
        bool bFirst = true;
        for (float step = 0; step < 2 * Mathf.PI; step += tempStep)
        {
            float x = distance * Mathf.Cos(step);
            float z = distance * Mathf.Sin(step);
            endPoint.x = selfPosition.x + x;
            endPoint.z = selfPosition.z + z;
 
            if (bFirst)
                bFirst = false;
            else
                Debug.DrawLine(beginPoint, endPoint, Color.red);
            
            beginPoint = endPoint;
        }
 
        //---------------------范围检测-----------------------------------
 
        float currDistance = Vector3.Distance(selfPosition, targetPosition);
        if (currDistance <= distance)
            return true;
 
        return false;
    }
 
    /// <summary>
    /// 三角形范围检测
    /// </summary>
    private bool TriangleCheck(Transform self, Transform target, float halfWidth,float distance)
    {
        if (null == self || null == target)
            return false;
 
        //---------------------绘制图形-----------------------------------
 
        Vector3 selfPosition = NormalizePosition(self.position);
        Vector3 targetPosition = NormalizePosition(target.position);
        Quaternion tempQuat = self.rotation;
 
        //三角形的三个点
        Vector3 leftPoint = selfPosition + (tempQuat * Vector3.left) * halfWidth;
        Vector3 rightPoint = selfPosition + (tempQuat * Vector3.right) * halfWidth;
        Vector3 forwardPoint = selfPosition + (tempQuat * Vector3.forward) * distance;
 
        Debug.DrawLine(leftPoint,rightPoint,Color.red);
        Debug.DrawLine(rightPoint, forwardPoint, Color.red);
        Debug.DrawLine(forwardPoint, leftPoint, Color.red);
 
        //---------------------范围检测-----------------------------------
 
        bool bResult = IsPointInTriangle(leftPoint, forwardPoint, rightPoint, targetPosition);
        return bResult;
    }
 
    /// <summary>
    /// 扇形范围检测
    /// </summary>
    private bool FanshapedCheck(Transform self, Transform target, float halfAngle, float distance)
    {
        if (null == self || null == target)
            return false;
 
        //---------------------绘制图形-----------------------------------
 
        Vector3 selfPosition = NormalizePosition(self.position);
        Vector3 targetPosition = NormalizePosition(target.position);
        Quaternion selfQuat = self.rotation;
 
        int nCircleDentity = 360;
        Vector3 firstPoint = Vector3.zero;
        Vector3 beginPoint = selfPosition;
        Vector3 endPoint = Vector3.zero;
        float tempStep = 2 * Mathf.PI / nCircleDentity;
        float leftRadian =  Mathf.PI / 2  + Mathf.Deg2Rad * halfAngle;
        float rightRadian = Mathf.PI / 2 - Mathf.Deg2Rad * halfAngle;
        bool bFirst = true;
        for (float step = 0; step < 2 * Mathf.PI; step += tempStep)
        {
            float x = distance * Mathf.Cos(step);
            float z = distance * Mathf.Sin(step);
            endPoint.x = selfPosition.x + x;
            endPoint.z = selfPosition.z + z;
 
            if (step >= rightRadian && step <= leftRadian)
            {
                if (bFirst)
                {
                    firstPoint = endPoint;
                    bFirst = false;
                }
                Debug.DrawLine(beginPoint, endPoint, Color.red);
 
                beginPoint = endPoint;
            }
        }
        Debug.DrawLine(selfPosition, firstPoint, Color.red);
        Debug.DrawLine(selfPosition, beginPoint, Color.red);
 
        //---------------------范围检测-----------------------------------
 
        //计算距离
        float currDis = Vector3.Distance(selfPosition, targetPosition);
        if (currDis > distance)
            return false;
 
        //计算self到target的向量
        Vector3 dir = targetPosition - selfPosition;
 
        //点乘dir向量和自身的forward向量 cosq
        float dotForward = Vector3.Dot(dir.normalized, (selfQuat * Vector3.forward).normalized);
 
        //得到夹角弧度并转换成角度
        float radian = Mathf.Acos(dotForward);
        float currAngle = Mathf.Rad2Deg * radian;
 
        if (Mathf.Abs(currAngle) <= halfAngle)
            return true;
 
        return false;
    }
 
    /// <summary>
    ///  矩形范围检测(数学点和矩形关系判断)
    /// </summary>
    private bool SimulateRectangleCheck(Transform self, Transform target, float halfWidth, float distance)
    {
        if (null == self || null == target)
            return false;
 
        //---------------------绘制图形-----------------------------------
 
        Vector3 selfPosition = NormalizePosition(self.position);
        Vector3 targetPosition = NormalizePosition(target.position);
        Vector3 selfEulerAngles = self.rotation.eulerAngles;
        Quaternion selfQuat = self.rotation;
 
        //矩形的四个点
        Vector3 leftPoint = selfPosition + (selfQuat * Vector3.left) * halfWidth;
        Vector3 rightPoint = selfPosition + (selfQuat * Vector3.right) * halfWidth;
        Vector3 leftUpPoint = leftPoint + (selfQuat * Vector3.forward) * distance;
        Vector3 rightUpPoint = rightPoint + (selfQuat * Vector3.forward) * distance;
 
        Debug.DrawLine(selfPosition, leftPoint, Color.red);
        Debug.DrawLine(selfPosition, rightPoint, Color.red);
        Debug.DrawLine(leftPoint, leftUpPoint, Color.red);
        Debug.DrawLine(rightPoint, rightUpPoint, Color.red);
        Debug.DrawLine(leftUpPoint, rightUpPoint, Color.red);
 
        //---------------------范围检测-----------------------------------
 
        Vector2 point = Vector2.zero;
        point.x = targetPosition.x;
        point.y = targetPosition.z;
 
        Vector2 point1 = Vector2.zero;
        point1.x = leftUpPoint.x;
        point1.y = leftUpPoint.z;
 
        Vector2 point2 = Vector2.zero;
        point2.x = rightUpPoint.x;
        point2.y = rightUpPoint.z;
 
        Vector2 point3 = Vector2.zero;
        point3.x = rightPoint.x;
        point3.y = rightPoint.z;
 
        Vector2 point4 = Vector2.zero;
        point4.x = leftPoint.x;
        point4.y = leftPoint.z;
 
        bool bResult = IsPointInRectangle(point1, point2, point3, point4, point);
        return bResult;
    }
 
    /// <summary>
    /// 矩形范围检测(点乘方式)
    /// </summary>
    private bool RectangleCheck(Transform self, Transform target, float halfWidth, float distance)
    {
        if (null == self || null == target)
            return false;
 
        //---------------------绘制图形-----------------------------------
 
        Vector3 selfPosition = NormalizePosition(self.position);
        Vector3 targetPosition = NormalizePosition(target.position);
        Vector3 selfEulerAngles = self.rotation.eulerAngles;
        Quaternion selfQuat = self.rotation;
 
        //矩形的四个点
        Vector3 leftPoint = selfPosition + (selfQuat * Vector3.left) * halfWidth;
        Vector3 rightPoint = selfPosition + (selfQuat * Vector3.right) * halfWidth;
        Vector3 leftUpPoint = leftPoint + (selfQuat * Vector3.forward) * distance;
        Vector3 rightUpPoint = rightPoint + (selfQuat * Vector3.forward) * distance;
 
        Debug.DrawLine(selfPosition, leftPoint, Color.red);
        Debug.DrawLine(selfPosition, rightPoint, Color.red);
        Debug.DrawLine(leftPoint, leftUpPoint, Color.red);
        Debug.DrawLine(rightPoint, rightUpPoint, Color.red);
        Debug.DrawLine(leftUpPoint, rightUpPoint, Color.red);
 
        //---------------------范围检测-----------------------------------
 
        //计算self到target的向量
        Vector3 dir = targetPosition - selfPosition;
 
        //点乘dir向量和自身的forward向量
        float dotForward = Vector3.Dot(dir, (selfQuat * Vector3.forward).normalized);
 
        //target处于self的前方的height范围
        if (dotForward > 0 && dotForward <= distance)
        {
            float dotRight = Vector3.Dot(dir, (selfQuat * Vector3.right).normalized);
 
            //target处于self的左右halfWidth的范围
            if (Mathf.Abs(dotRight) <= halfWidth)
                return true;
        }
        return false;
    }
 
    /// <summary>
    /// 扇面范围检测
    /// </summary>
    private bool SectorCheck(Transform self, Transform target, float halfAngle, float nearDis, float farDis)
    {
        if (null == self || null == target)
            return false;
 
        if (nearDis > farDis)
        {
            float tempDis = nearDis;
            nearDis = farDis;
            farDis = tempDis;
        }
        //---------------------绘制图形-----------------------------------
 
        Vector3 selfPosition = NormalizePosition(self.position);
        Vector3 targetPosition = NormalizePosition(target.position);
        Quaternion selfQuat = self.rotation;
 
        int nCircleDentity = 360;
        Vector3 nearFirstPoint = Vector3.zero;
        Vector3 nearBeginPoint = selfPosition;
        Vector3 nearEndPoint = Vector3.zero;
        Vector3 farFirstPoint = Vector3.zero;
        Vector3 farBeginPoint = selfPosition;
        Vector3 farEndPoint = Vector3.zero;
        float tempStep = 2 * Mathf.PI / nCircleDentity;
        float leftRadian = Mathf.PI / 2 + Mathf.Deg2Rad * halfAngle;
        float rightRadian = Mathf.PI / 2 - Mathf.Deg2Rad * halfAngle;
        bool bFirst = true;
        for (float step = 0; step < 2 * Mathf.PI; step += tempStep)
        {
            float nearX = nearDis * Mathf.Cos(step);
            float nearZ = nearDis * Mathf.Sin(step);
 
            float farX = farDis * Mathf.Cos(step);
            float farZ = farDis * Mathf.Sin(step);
 
            if (step >= rightRadian && step <= leftRadian)
            {
                //-------绘制近扇面
                nearEndPoint.x = selfPosition.x + nearX;
                nearEndPoint.z = selfPosition.z + nearZ;
 
                //-------绘制远扇面
                farEndPoint.x = selfPosition.x + farX;
                farEndPoint.z = selfPosition.z + farZ;
 
                if (bFirst)
                {
                    nearFirstPoint = nearEndPoint;
                    farFirstPoint = farEndPoint;
                    bFirst = false;
                }
                else
                {
                    Debug.DrawLine(nearBeginPoint, nearEndPoint, Color.red);
                    Debug.DrawLine(farBeginPoint, farEndPoint, Color.red);
                }
                    
                nearBeginPoint = nearEndPoint;
                farBeginPoint = farEndPoint;
            }
        }
        Debug.DrawLine(nearFirstPoint, farFirstPoint, Color.red);
        Debug.DrawLine(nearEndPoint, farEndPoint, Color.red);
        Debug.DrawLine(selfPosition, nearFirstPoint, Color.blue);
        Debug.DrawLine(selfPosition, nearEndPoint, Color.blue);
 
        //---------------------范围检测-----------------------------------
 
        //计算距离
        float currDis = Vector3.Distance(selfPosition, targetPosition);
        if (currDis < nearDis ||  currDis > farDis)
            return false;
 
        //计算self到target的向量
        Vector3 dir = targetPosition - selfPosition;
 
        //点乘dir向量和自身的forward向量 cosq
        float dotForward = Vector3.Dot(dir.normalized, (selfQuat * Vector3.forward).normalized);
 
        //得到夹角弧度并转换成角度
        float radian = Mathf.Acos(dotForward);
        float currAngle = Mathf.Rad2Deg * radian;
 
        if (Mathf.Abs(currAngle) <= halfAngle)
            return true;
 
        return false;
    }
 
    /// <summary>
    /// 双圆范围检测
    /// </summary>
    private bool RingCheck(Transform self, Transform target, float nearDis, float farDis)
    {
        if (null == self || null == target)
            return false;
 
        if (nearDis > farDis)
        {
            float tempDis = nearDis;
            nearDis = farDis;
            farDis = tempDis;
        }
        //---------------------绘制图形-----------------------------------
 
        Vector3 selfPosition = NormalizePosition(self.position);
        Vector3 targetPosition = NormalizePosition(target.position);
 
        int nCircleDentity = 360;
        Vector3 nearBeginPoint = selfPosition;
        Vector3 nearEndPoint = Vector3.zero;
        Vector3 farBeginPoint = selfPosition;
        Vector3 farEndPoint = Vector3.zero;
        float tempStep = 2 * Mathf.PI / nCircleDentity;
        bool bFirst = true;
        for (float step = 0; step < 2 * Mathf.PI; step += tempStep)
        {
            float nearX = nearDis * Mathf.Cos(step);
            float nearZ = nearDis * Mathf.Sin(step);
            nearEndPoint.x = selfPosition.x + nearX;
            nearEndPoint.z = selfPosition.z + nearZ;
 
            float farX = farDis * Mathf.Cos(step);
            float farZ = farDis * Mathf.Sin(step);
            farEndPoint.x = selfPosition.x + farX;
            farEndPoint.z = selfPosition.z + farZ;
 
            if (bFirst)
                bFirst = false;
            else
            {
                Debug.DrawLine(nearBeginPoint, nearEndPoint, Color.red);
                Debug.DrawLine(farBeginPoint, farEndPoint, Color.red);
            }
 
            nearBeginPoint = nearEndPoint;
            farBeginPoint = farEndPoint;
        }
 
        //---------------------范围检测-----------------------------------
 
        float currDistance = Vector3.Distance(selfPosition, targetPosition);
        if (currDistance >= nearDis && currDistance <= farDis )
            return true;
 
        return false;
    }
 
    /// <summary>
    /// 规范位置(去除高度带来的影响)
    /// </summary>
    private Vector3 NormalizePosition(Vector3 position,float hight = 0.0f)
    {
        Vector3 tempPosition = Vector3.zero;
        tempPosition.x = position.x;
        tempPosition.y = hight;
        tempPosition.z = position.z;
 
        return tempPosition;
    }
 
    /// <summary>
    /// 三角形检查
    /// </summary>
    private bool IsPointInTriangle(Vector3 point1, Vector3 point2, Vector3 point3, Vector3 targetPoint)
    {
        Vector3 v0 = point2 - point1;
        Vector3 v1 = point3 - point1;
        Vector3 v2 = targetPoint - point1;
 
        float dot00 = Vector3.Dot(v0, v0);
        float dot01 = Vector3.Dot(v0, v1);
        float dot02 = Vector3.Dot(v0, v2);
        float dot11 = Vector3.Dot(v1, v1);
        float dot12 = Vector3.Dot(v1, v2);
 
        float inverDeno = 1 / (dot00 * dot11 - dot01 * dot01);
        float u = (dot11 * dot02 - dot01 * dot12) * inverDeno;
        if (u < 0 || u > 1)
            return false;
 
        float v = (dot00 * dot12 - dot01 * dot02) * inverDeno;
        if (v < 0 || v > 1)
            return false;
 
        return u + v <= 1;
    }
 
    /// <summary>
    /// 判断点p是否在p1 p2 p3 p4构成的矩形内
    /// </summary>
    private bool IsPointInRectangle(Vector2 point1, Vector2 point2, Vector2 point3, Vector2 point4, Vector2 point)
    {
        return GetCross(point1, point2, point) * GetCross(point3, point4, point) >= 0
            && GetCross(point2, point3, point) * GetCross(point4, point1, point) >= 0;
    }
 
    /// <summary>
    /// 计算 |p1 p2| X |p1 p|
    /// </summary>
    private float GetCross(Vector2 point1, Vector2 point2, Vector2 point)
    {
        return ((point2.x - point1.x) * (point.y - point1.y) - (point.x - point1.x) * (point2.y - point1.y));
    }
}
