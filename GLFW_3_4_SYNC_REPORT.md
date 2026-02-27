# Follow-up Commit Review: "Address GLFW 3.4 sync review discrepancies"

**Commit**: `9b4d7c3` · **Files changed**: 4 · **+59 / -0**

## Original Issues → Resolution

| # | Original Issue | Status | Notes |
|---|---|---|---|
| 1 | Missing `panicError()` in `InitVulkanLoader` | ✅ Fixed | Added at line 57 of `vulkan.go` |
| 2 | `glfwGetError` not wrapped | ✅ Fixed | New `GetError()` function in `error.go` with `NoError` constant |
| 3 | `glfwGetPhysicalDevicePresentationSupport` not wrapped | ✅ Fixed | New function in `vulkan.go` following `CreateWindowSurface` patterns |
| 4 | `GLFW_UNLIMITED_MOUSE_BUTTONS` missing | ✅ Addressed | README notes it's absent from the vendored GLFW C revision |
| 5 | `GetTitle` doc comment | ✅ Fixed | Added copy-semantics note |
| 6 | README cumulative changelog | ✅ Fixed | Added clarifying note at section top |

## New Issue Found

### Variable Shadowing in `GetPhysicalDevicePresentationSupport`

In [vulkan.go line 84](file:///home/alex/test/go-gl/gogl/v3.4/glfw/vulkan.go#L84), the short variable declaration `err :=` **shadows** the named return parameter `err`:

```go
func GetPhysicalDevicePresentationSupport(instance, device interface{}, queueFamily uint32) (supported bool, err error) {
    // ...
    if err := acceptError(APIUnavailable); err != nil {  // ← shadows named return 'err'
        return false, err
    }
    // ...
}
```

This is **functionally correct** — the inner `err` is used correctly within the `if` block and the intended error is returned. However, it will trigger `go vet` / linter warnings for variable shadowing, and it's inconsistent with the rest of the codebase. Changing `err :=` to `err =` would use the named return directly and silence the warning.

## Overall Verdict

All original review items have been properly addressed. The implementations are clean and follow existing codebase conventions. The `GetError` function smartly drains the internal error channel to stay in sync with the callback-based error mechanism. The `GetPhysicalDevicePresentationSupport` correctly mirrors the `CreateWindowSurface` pattern for Vulkan type handling.

**Only the minor shadowing issue above remains.**
