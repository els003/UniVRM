`0.82.1` の API です。
`0.82.0` の場合は更新をお願いします。

以下のステップでロードします。

1. `GLB` / `GLTF` をパースして `GltfData` を得る。
2. `GltfData` から `Unity Hierarchy` を ロード する。`RuntimeGltfInstance` を得る。 ローダーを破棄する。
3. ロードした `RuntimeGltfInstance` 使う。`RuntimeGltfInstance` を破棄する。

# 1. パースする

## glb ファイルパスからパースする

* `vrm` もこの関数を使います。

```cs
GltfData Load(string path)
{
    return new GlbFileParser(path).Parse();
}
```

## glb バイト列をパースする

* `vrm` もこの関数を使います。

```cs
GltfData Load(byte[] bytes)
{
    return new GlbBinaryParser(bytes, "LOAD_NAME").parse();
}
```

## gltf ファイルパスからパースする

```cs
GltfData Load(string path)
{
    return new GltfFileWithResourceFilesParser(path).Parse();
}
```

## zip アーカイブからパースする

gltf と関連するファイルを zip アーカイブしたファイルをパースできます(実験)。

```cs
GltfData Load(string path)
{
    return new ZipArchivedGltfFileParser(path).Parse();
}
```

## ファイルパスの拡張子でパースする

サンプルの `SimpleViewer` を参考にしてください。

```cs
GltfData Load(string path)
{
    // ファイル拡張子で自動判定します
    return new AutoGltfFileParser(path).Parse();
}
```

# 2. ロードする

## sync

```cs
RuntimeGltfInstance Load(GltfData data)
{
    // ImporterContext は使用後に Dispose を呼び出してください。
    // using で自動的に呼び出すことができます。
    using(var loader = new UniGLTF.ImporterContext(data)
    {
        var instance = loader.Load();
        return instance;
    }
}
```

## async

```cs
async RuntimeGltfInstance Load(GltfData data)
{
    // ImporterContext は使用後に Dispose を呼び出してください。
    // using で自動的に呼び出すことができます。
    using(var loader = new UniGLTF.ImporterContext(data)
    {
        var instance = await loader.LoadAsync();
        return instance;
    }
}
```

## materialGenerator で URP 用のマテリアルをロードする

`materialGenerator` 引き数(省略可能)を指定することで URP マテリアルを生成するようにカスタムできます。

```cs
async RuntimeGltfInstance Load(GltfData data)
{
    var materialGenerator = new GltfUrpMaterialDescriptorGenerator();
    using(var loader = new UniGLTF.ImporterContext(data, materialGenerator: materialGenerator)
    {
        var instance = await loader.LoadAsync();
        return instance;
    }
}
```

# 3. インスタンスを使用する

```cs
// SkinnedMeshRenderer に対する指示
instance.EnableUpdateWhenOffscreen();
// 準備ができたら表示する(デフォルトでは非表示)
instance.ShowMeshes();
```

使用後に以下のように破棄してください。関連する Asset(Texture, Material, Meshなど)も破棄されます。
```cs
GameObject.Destroy(instance);
```
