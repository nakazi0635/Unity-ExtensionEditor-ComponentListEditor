# Unity-ExtensionEditor-ComponentListEditor
アセットにあるオブジェクトのコンポーネントを一覧表示・検索・追加するクラス

初期状態はParticleSystem,Rigidbody,BoxColliderのフィルターが可能

自由にフィルターを追加することが可能  
例：17,89,90行目をコメントアウトすると、Canvasコンポーネントのフィルターが追加される

## 内容

```cs
using UnityEngine;
using UnityEditor;
using System;

// アセットにあるオブジェクトのコンポーネントを一覧表示・検索・追加するクラス
public class ComponentListEditor : EditorWindow
{

    private GameObject selectedPrefab;
    private Vector2 scrollPosition;
    // 選択可能なコンポーネントタイプ、enumを使用
    private enum ComponentType{
        All,
        ParticleSystem,
        Rigidbody,
        BoxCollider,
        // Canvas
        // 他のコンポーネントを追加する場合はここに列挙
    }
    private ComponentType selectedComponentType = ComponentType.All;

    // メニュー欄に追加
    [MenuItem("エディター拡張/プレハブ一覧・検索・追加")]
    public static void ShowWindow(){
        // エディターウィンドウを作成して表示し、そのインスタンスを返す
        EditorWindow.GetWindow(typeof(ComponentListEditor));
    }
    
    // ウィンドウ上のレイアウト
    private void OnGUI(){
        // ラベルの表示
        GUILayout.Label("プレハブ一覧・検索・追加", EditorStyles.boldLabel);
        // ポップアップを表示して、選択されたコンポーネントを変数に代入
        selectedComponentType = (ComponentType)EditorGUILayout.EnumPopup("コンポーネント", selectedComponentType);
        // スクロール可能範囲の開始
        scrollPosition = EditorGUILayout.BeginScrollView(scrollPosition);

        // GetPrefabsWithComponentメソッドから取得したプレハブの一覧をループ処理し、各プレハブの名前をボタンとして表示
        foreach (var prefab in GetPrefabsWithComponent(selectedComponentType)){
            if (GUILayout.Button(prefab.name, GUILayout.Width(position.width - 20))){
                // 選択されたprefab内のオブジェクトを代入し、CreatePrefabメソッドに移動
                selectedPrefab = prefab;
                CreatePrefab();
            }
        }
        // スクロール可能範囲の終了
        EditorGUILayout.EndScrollView();
    }

    // プロジェクト内の全てのプレハブを検索し、選択されたコンポーネントを持つプレハブを返すメソッド
    private GameObject[] GetPrefabsWithComponent(ComponentType componentType){
        // プロジェクト内の全てのデータからプレハブのGUIDを取得
        string[] guids = AssetDatabase.FindAssets("t:Prefab");
        //　取得した数だけ要素を取得
        GameObject[] prefabs = new GameObject[guids.Length];
        for (int i = 0; i < guids.Length; i++){
            // 取得したGUIDからパスを特定
            string path = AssetDatabase.GUIDToAssetPath(guids[i]);
            // 特定したパスから、プレハブの実際のGameObjectを取得
            prefabs[i] = AssetDatabase.LoadAssetAtPath<GameObject>(path);
        }
        // Allが選択されていたらフィルタリングせずに返す
        if (componentType == ComponentType.All){
            return prefabs;
        }else{
            // 選択されたコンポーネントを持つプレハブを調べる
            Type type = GetComponentType(componentType);
            if (type != null){
                // prefabs配列内のプレハブの中から、選択されたコンポーネントを持つプレハブのみをフィルタリング
                return System.Array.FindAll(prefabs, prefab => prefab.GetComponent(type) != null);
            }else{
                // 一つもなければ空の配列を返す
                return new GameObject[0];
            }
        }
    }

    // ComponentTypeに基づいて対応するコンポーネントのTypeオブジェクトを取得するメソッド
    private Type GetComponentType(ComponentType componentType){
        // 判定しType型を返す
        switch (componentType){
            case ComponentType.ParticleSystem:
                return typeof(ParticleSystem);
            case ComponentType.Rigidbody:
                return typeof(Rigidbody);
            case ComponentType.BoxCollider:
                return typeof(BoxCollider);

            // case ComponentType.Canvas:
            //     return typeof(Canvas);

            // 追加したコンポーネントに合わせてここに追加
            // 例えばCanvasを追加したい場合は上記をコメントアウト

            // 該当しない場合nullを返す
            default:
                return null;
        }
    }

    // 選択されたprefab内のオブジェクトをヒエラルキーに追加するメソッド
    private void CreatePrefab(){
        if (selectedPrefab == null) return;

        // プレハブをインスタンス化
        GameObject Prefab = PrefabUtility.InstantiatePrefab(selectedPrefab) as GameObject;
        // インスタンス化したオブジェクトをヒエラルキーウィンドウで選択状態に変更
        Selection.activeObject = Prefab;
        // インスタンス化したオブジェクトをヒエラルキーウィンドウで強調表示
        EditorGUIUtility.PingObject(Prefab);
    }
}
```