# RecyclerView

## 上拉加载更多

```java
//监听RecyclerView滑动，实现上拉加载更多
private RecyclerView.OnScrollListener mOnScrollListener = new RecyclerView.OnScrollListener() {
        private int lastVisibleItem;
        @Override
        public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
            super.onScrollStateChanged(recyclerView, newState);
            if (newState == RecyclerView.SCROLL_STATE_IDLE
                    && lastVisibleItem == adapter.getItemCount()){
                //延迟发送，模拟
                showProgress();
                myHandler.sendEmptyMessageDelayed(REFRESH_COMPLETE,2000);
            }
        }

        @Override
        public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
            super.onScrolled(recyclerView, dx, dy);
            lastVisibleItem = mLayoutManager.findLastVisibleItemPosition();s
        }
    };
```

## 下拉刷新

```java
@Override
    public void onRefresh() {
        showProgress();
     myHandler.sendEmptyMessageDelayed(REFRESH_LRARN,2000);
    }
```

## ViewPager

```java
package com.chunsoft.csdn_example;
import android.content.Context;
import android.os.Bundle;
import android.os.Handler;
import android.support.v4.view.ViewPager;
import android.support.v4.widget.SwipeRefreshLayout;
import android.support.v7.app.AppCompatActivity;
import android.support.v7.widget.DefaultItemAnimator;
import android.support.v7.widget.LinearLayoutManager;
import android.support.v7.widget.RecyclerView;
import android.util.Log;
import android.view.LayoutInflater;
import android.view.MotionEvent;
import android.view.View;
import android.view.ViewGroup;
import android.widget.LinearLayout;
import android.widget.Toast;

import java.lang.ref.WeakReference;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class MainActivity extends AppCompatActivity implements SwipeRefreshLayout.OnRefreshListener{
    private static final int REFRESH_COMPLETE = 0X110;
    private static final int REFRESH_LRARN = 0X111;

    private SwipeRefreshLayout mSwipeRefreshLayout;
    private XRecycleView mRecyclerView;


    //Add Header
    private View mHeadViewer;
    private DecoratorViewPager mPager;
    private LinearLayoutManager mLayoutManager;
    private LinearLayout mLlDot;
    private ArrayList mImgList;
    private int curIndex = 0;
    private ViewPagerAdapter myPagerAdapter;

    private HomeAdapter adapter;
    private List<String> mDatas;

    public void showProgress() {
        mSwipeRefreshLayout.setRefreshing(true);
    }


    public void hideProgress() {
        mSwipeRefreshLayout.setRefreshing(false);
        adapter.isShowFooter(false);
    }

    private MyHandler myHandler = new MyHandler(this);

    private static class MyHandler extends Handler {
        private WeakReference<Context> weakReference;  //用弱引用防止造成OOM

        public  MyHandler(Context context){
            weakReference = new WeakReference<>(context);
        }
        public void handleMessage(android.os.Message msg) {

            MainActivity activity = (MainActivity)weakReference.get();
            switch (msg.what)
            {
                case REFRESH_COMPLETE:
                    if(activity!=null){
                        activity.mDatas.addAll(Arrays.asList("上拉数据1", "上拉数据2", "上拉数据3"));
                        activity.adapter.notifyDataSetChanged();
                    }
                    activity.hideProgress();

                    break;
                case  REFRESH_LRARN:
                    if(activity!=null){
                        activity.mDatas.addAll(Arrays.asList("下拉数据1", "下拉数据2", "下拉数据3"));
                        activity.adapter.notifyDataSetChanged();
                        activity.mSwipeRefreshLayout.setRefreshing(false);
                    }
                    activity.hideProgress();
                    break;
            }
        }
    }

    private RecyclerView.OnScrollListener mOnScrollListener = new RecyclerView.OnScrollListener() {
        private int lastVisibleItem;
        @Override
        public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
            super.onScrollStateChanged(recyclerView, newState);
            if (newState == RecyclerView.SCROLL_STATE_IDLE
                    && lastVisibleItem == adapter.getItemCount()){
                //延迟发送，模拟
                showProgress();
                myHandler.sendEmptyMessageDelayed(REFRESH_COMPLETE,2000);
            }
        }

        @Override
        public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
            super.onScrolled(recyclerView, dx, dy);
            lastVisibleItem = mLayoutManager.findLastVisibleItemPosition();
        }
    };



    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mSwipeRefreshLayout = (SwipeRefreshLayout) findViewById(R.id.refresh);
        //设置swipe加载样式
        mSwipeRefreshLayout.setColorSchemeColors(
                getResources().getColor(android.R.color.holo_blue_bright),
                getResources().getColor(android.R.color.holo_green_light),
                getResources().getColor(android.R.color.holo_orange_light),
                getResources().getColor(android.R.color.holo_red_light));
        mRecyclerView = (XRecycleView) findViewById(R.id.recycle_view);

        initData();
        initView();
        initEvent();

    }

    private void initEvent() {
        mSwipeRefreshLayout.setOnRefreshListener(this);
        mRecyclerView.addOnScrollListener(mOnScrollListener);
        mRecyclerView.setAdapter(adapter);
        mPager.setAdapter(myPagerAdapter);
        adapter.setOnItemClickListtener(mOnItemClickListtener);

    }

    private void initView()
    {
        mHeadViewer = LayoutInflater.from(this).inflate(R.layout.home_header,null);

        //顶部ViewPager
        mPager = (DecoratorViewPager) mHeadViewer.findViewById(R.id.viewpager);
        mPager.setNestedpParent((ViewGroup)mPager.getParent());

        //屏蔽ViewPager事件和SwipeRefreshLayout事件冲突
        mPager.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                switch (event.getAction()) {
                    case MotionEvent.ACTION_MOVE:
                        mSwipeRefreshLayout.setEnabled(false);
                        break;
                    case MotionEvent.ACTION_UP:
                    case MotionEvent.ACTION_CANCEL:
                        mSwipeRefreshLayout.setEnabled(true);
                        break;
                }
                return false;
            }
        });
        mLlDot = (LinearLayout) mHeadViewer.findViewById(R.id.ll_dot);

        setOvalLayout();
        //mRecyclerView.setHasFixedSize(true);
        mLayoutManager = new LinearLayoutManager(this);
        mRecyclerView.setLayoutManager(mLayoutManager);
        mRecyclerView.setItemAnimator(new DefaultItemAnimator());
        mRecyclerView.addHeaderView(mHeadViewer);
        adapter = new HomeAdapter(this,mDatas);
        myPagerAdapter = new ViewPagerAdapter(this,mImgList);

    }

    private void initData()
    {
        mDatas = new ArrayList<>(Arrays.asList("C", "C++", "HTML", "JAVA", "Objective-C",
                "javascript", "JSP/servelet", "ASP.net", "数据结构", "Oracle"));
        mImgList = new ArrayList();
        mImgList.add(R.drawable.gao0);
        mImgList.add(R.drawable.gao1);
        mImgList.add(R.drawable.gao2);
        mImgList.add(R.drawable.gao3);
        mImgList.add(R.drawable.gao4);
    }

    /**
     * 设置圆点
     */
    private void setOvalLayout(){
        for(int i = 0;i < 5;i++){
            mLlDot.addView(LayoutInflater.from(this).inflate(R.layout.dot,null));
        }
        // 默认显示第一页
        mLlDot.getChildAt(0).findViewById(R.id.v_dot)
                .setBackgroundResource(R.drawable.dot_selected);
        mPager.setOnPageChangeListener(new ViewPager.OnPageChangeListener(){

            @Override
            public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {

            }

            @Override
            public void onPageSelected(int position) {
                //取消圆点选中
                mLlDot.getChildAt(curIndex)
                        .findViewById(R.id.v_dot)
                        .setBackgroundResource(R.drawable.dot_normal);
                mLlDot.getChildAt(position)
                        .findViewById(R.id.v_dot)
                        .setBackgroundResource(R.drawable.dot_selected);
                curIndex = position;

            }

            @Override
            public void onPageScrollStateChanged(int state) {

            }
        });
    }
    @Override
    public void onRefresh() {
        showProgress();
        myHandler.sendEmptyMessageDelayed(REFRESH_LRARN,2000);
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        myHandler.removeCallbacksAndMessages(null);
    }
    private HomeAdapter.OnItemClickListener mOnItemClickListtener = new HomeAdapter.OnItemClickListener() {
        @Override
        public void onItemClick(View view, int position) {
            Toast.makeText(view.getContext(),"点击："+position,Toast.LENGTH_LONG).show();
            Log.e("---------->","点击："+position);
        }
    };
}

```

## ViewPager左右滑动以及SwipeRefreshLayout上下滑动冲突

```java
 //顶部ViewPager
        mPager = (DecoratorViewPager) mHeadViewer.findViewById(R.id.viewpager);
        mPager.setNestedpParent((ViewGroup)mPager.getParent());

        //屏蔽ViewPager事件和SwipeRefreshLayout事件冲突
        mPager.setOnTouchListener(new View.OnTouchListener() {
            @Override
            public boolean onTouch(View v, MotionEvent event) {
                switch (event.getAction()) {
                    case MotionEvent.ACTION_MOVE:
                        mSwipeRefreshLayout.setEnabled(false);
                        break;
                    case MotionEvent.ACTION_UP:
                    case MotionEvent.ACTION_CANCEL:
                        mSwipeRefreshLayout.setEnabled(true);
                        break;
                }
                return false;
            }
        });
```

