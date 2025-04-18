Menu
Using App Router
Features available in /app
Using Latest Version
15.2.0
Using App Router
Features available in /app
Using Latest Version
15.2.0
API ReferenceFunctionsgenerateImageMetadata
# generateImageMetadata
You can use `generateImageMetadata` to generate different versions of one image or return multiple images for one route segment. This is useful for when you want to avoid hard-coding metadata values, such as for icons.
## Parameters
`generateImageMetadata` function accepts the following parameters:
#### `params` (optional)
An object containing the dynamic route parameters object from the root segment down to the segment `generateImageMetadata` is called from.
icon.tsx
TypeScript
JavaScriptTypeScript
```
exportfunctiongenerateImageMetadata({
 params,
}: {
 params: { slug:string }
}) {
// ...
}
```

Route| URL| `params`  
---|---|---  
`app/shop/icon.js`| `/shop`| `undefined`  
`app/shop/[slug]/icon.js`| `/shop/1`| `{ slug: '1' }`  
`app/shop/[tag]/[item]/icon.js`| `/shop/1/2`| `{ tag: '1', item: '2' }`  
## Returns
The `generateImageMetadata` function should return an `array` of objects containing the image's metadata such as `alt` and `size`. In addition, each item **must** include an `id` value which will be passed to the props of the image generating function.
Image Metadata Object| Type  
---|---  
`id`| `string` (required)  
`alt`| `string`  
`size`| `{ width: number; height: number }`  
`contentType`| `string`  
icon.tsx
TypeScript
JavaScriptTypeScript
```
import { ImageResponse } from'next/og'
exportfunctiongenerateImageMetadata() {
return [
  {
   contentType:'image/png',
   size: { width:48, height:48 },
   id:'small',
  },
  {
   contentType:'image/png',
   size: { width:72, height:72 },
   id:'medium',
  },
 ]
}
exportdefaultfunctionIcon({ id }: { id:string }) {
returnnewImageResponse(
  (
   <div
style={{
     width:'100%',
     height:'100%',
     display:'flex',
     alignItems:'center',
     justifyContent:'center',
     fontSize:88,
     background:'#000',
     color:'#fafafa',
    }}
   >
    Icon {id}
   </div>
  )
 )
}
```

### Examples
#### Using external data
This example uses the `params` object and external data to generate multiple Open Graph images for a route segment.
app/products/[id]/opengraph-image.tsx
TypeScript
JavaScriptTypeScript
```
import { ImageResponse } from'next/og'
import { getCaptionForImage, getOGImages } from'@/app/utils/images'
exportasyncfunctiongenerateImageMetadata({
 params,
}: {
 params: { id:string }
}) {
constimages=awaitgetOGImages(params.id)
returnimages.map((image, idx) => ({
  id: idx,
  size: { width:1200, height:600 },
  alt:image.text,
  contentType:'image/png',
 }))
}
exportdefaultasyncfunctionImage({
 params,
 id,
}: {
 params: { id:string }
 id:number
}) {
constproductId= (await params).id
constimageId= id
consttext=awaitgetCaptionForImage(productId, imageId)
returnnewImageResponse(
  (
   <div
style={
     {
// ...
     }
    }
   >
    {text}
   </div>
  )
 )
}
```

## Version History
Version| Changes  
---|---  
`v13.3.0`| `generateImageMetadata` introduced.  
## Next Steps
View all the Metadata API options.
### Metadata Files
API documentation for the metadata file conventions.
### Metadata
Use the Metadata API to define metadata in any layout or page.
Was this helpful?
supported.
Send
