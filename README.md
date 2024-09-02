***Toon Shader for Unity***

The HLSL file/script for creating a Toon Shader on Unity - interactives and reacts to lights, including point and directional lights. Has adjustable public values like: ramp offset, tinting, texture assignment, rim power, rim brightness, and ambient controls.
More information on its features here: https://www.malani.work/toon-shader

Here is a copy/paste version of the script for quick application:

```
void ToonShader_float(in float3 Normal, in float ToonRampSmoothness, in float3 ClipSpacePos, in float3 WorldPos, in float3 ToonRampTinting,
in float ToonRampOffset, in float ToonRampOffsetPoint, in float Ambient, out float3 ToonRampOutput, out float3 Direction)
{
    // Check if we're in the Shader Graph preview mode
    #ifdef SHADERGRAPH_PREVIEW
        // If so, set default values for ToonRampOutput and Direction
        ToonRampOutput = float3(0.5,0.5,0);
        Direction = float3(0.5,0.5,0);
    #else
 
        // Determine if we need screen space or world space shadows
        #if SHADOWS_SCREEN
            // Compute shadow coordinates from clip space position
            half4 shadowCoord = ComputeScreenPos(ClipSpacePos);
        #else
            // Compute shadow coordinates from world position
            half4 shadowCoord = TransformWorldToShadowCoord(WorldPos);
        #endif 

        // Check if we need to consider cascaded shadows or not
        #if _MAIN_LIGHT_SHADOWS_CASCADE || _MAIN_LIGHT_SHADOWS
            // Retrieve the main light considering shadows
            Light light = GetMainLight(shadowCoord);
        #else
            // Retrieve the main light without considering shadows
            Light light = GetMainLight();
        #endif
 
        // Compute the dot product of the normal and light direction, adjusting to a [0,1] range
        half d = dot(Normal, light.direction) * 0.5 + 0.5;
        
        // Apply a smoothstep function to determine the toon shading ramp based on the dot product
        half toonRamp = smoothstep(ToonRampOffset, ToonRampOffset + ToonRampSmoothness, d);
        
        // Initialize variable to accumulate additional light contributions
        float3 extraLights = float3(0, 0, 0);
        
        // Get the count of additional lights affecting the pixel
        int pixelLightCount = GetAdditionalLightsCount();
        
        // Loop through each additional light
        for (int j = 0; j < pixelLightCount; ++j) {
            // Retrieve the additional light
            Light aLight = GetAdditionalLight(j, WorldPos, half4(1, 1, 1, 1));
            
            // Compute the color of the light considering distance and shadow attenuation
            float3 attenuatedLightColor = aLight.color * (aLight.distanceAttenuation * aLight.shadowAttenuation);
            
            // Compute the dot product of the normal and this additional light's direction, adjusting to [0,1] range
            half d = dot(Normal, aLight.direction) * 0.5 + 0.5;
            
            // Apply the toon shading ramp to the additional light based on its dot product
            half toonRampExtra = smoothstep(ToonRampOffsetPoint, ToonRampOffsetPoint + ToonRampSmoothness, d);
 
            // Accumulate the contribution of this additional light to the extraLights variable
            extraLights += (attenuatedLightColor * toonRampExtra);
        }
        
        // Adjust the main toon ramp output by shadow attenuation
        toonRamp *= light.shadowAttenuation;
 
        // Compute the final toon ramp output color by combining light color, toon ramp, tinting, and ambient light
        ToonRampOutput = light.color * (toonRamp + ToonRampTinting)  + Ambient;
 
        // Add contributions from additional lights to the final toon ramp output
        ToonRampOutput += extraLights;
        
        // Check if the main light is considered
        #if MAIN_LIGHT
            // Set the direction output to the normalized direction of the main light
            Direction = normalize(light.direction);
        #else
            // Set a default direction value if the main light is not considered
            Direction = float3(0.5,0.5,0);
        #endif
 
    #endif
}
```
